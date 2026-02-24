---
summary: "영구 상태를 가진 GCP Compute Engine VM (Docker)에서 OpenClaw 게이트웨이를 24/7 실행"
read_when:
  - GCP에서 OpenClaw을 24/7 실행하고 싶을 때
  - 자체 VM에서 프로덕션급 상시 가동 게이트웨이를 원할 때
  - 영속성, 바이너리, 재시작 동작에 대한 완전한 제어를 원할 때
title: "GCP"
---

# GCP Compute Engine에서 OpenClaw 실행 (Docker, 프로덕션 VPS 가이드)

## 목표

GCP Compute Engine VM에서 Docker를 사용하여 영구 상태, 내장 바이너리, 안전한 재시작 동작을 갖춘 지속적인 OpenClaw 게이트웨이를 실행합니다.

"월 ~$5-12에 OpenClaw 24/7"을 원한다면, 이것이 Google Cloud에서 신뢰할 수 있는 설정입니다.
가격은 머신 유형과 리전에 따라 다릅니다; 워크로드에 맞는 가장 작은 VM을 선택하고 OOM이 발생하면 확장하세요.

## 무엇을 하는 건가요 (쉽게 설명)?

- GCP 프로젝트 생성 및 결제 활성화
- Compute Engine VM 생성
- Docker 설치 (격리된 앱 런타임)
- Docker에서 OpenClaw 게이트웨이 시작
- 호스트에 `~/.openclaw` + `~/.openclaw/workspace` 영속화 (재시작/재빌드에도 유지)
- SSH 터널을 통해 노트북에서 Control UI 접근

게이트웨이는 다음을 통해 접근할 수 있습니다:

- 노트북에서 SSH 포트 포워딩
- 방화벽과 토큰을 직접 관리하는 경우 직접 포트 노출

이 가이드는 GCP Compute Engine의 Debian을 사용합니다.
Ubuntu도 작동합니다; 패키지를 적절히 매핑하세요.
일반적인 Docker 흐름은 [Docker](/install/docker)를 참조하세요.

---

## 빠른 경로 (경험 있는 운영자)

1. GCP 프로젝트 생성 + Compute Engine API 활성화
2. Compute Engine VM 생성 (e2-small, Debian 12, 20GB)
3. VM에 SSH 접속
4. Docker 설치
5. OpenClaw 저장소 클론
6. 영구 호스트 디렉토리 생성
7. `.env` 및 `docker-compose.yml` 구성
8. 필요한 바이너리 내장, 빌드 및 실행

---

## 필요한 것

- GCP 계정 (e2-micro 무료 티어 대상)
- gcloud CLI 설치 (또는 Cloud Console 사용)
- 노트북에서 SSH 접근
- SSH + 복사/붙여넣기에 대한 기본적인 익숙함
- 약 20-30분
- Docker 및 Docker Compose
- 모델 인증 자격 증명
- 선택적 프로바이더 자격 증명
  - WhatsApp QR
  - Telegram 봇 토큰
  - Gmail OAuth

---

## 1) gcloud CLI 설치 (또는 Console 사용)

**옵션 A: gcloud CLI** (자동화에 권장)

[https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)에서 설치

초기화 및 인증:

```bash
gcloud init
gcloud auth login
```

**옵션 B: Cloud Console**

모든 단계는 [https://console.cloud.google.com](https://console.cloud.google.com)의 웹 UI에서 수행할 수 있습니다

---

## 2) GCP 프로젝트 생성

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

[https://console.cloud.google.com/billing](https://console.cloud.google.com/billing)에서 결제를 활성화합니다 (Compute Engine에 필요).

Compute Engine API 활성화:

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1. IAM & 관리자 > 프로젝트 만들기로 이동
2. 이름을 지정하고 생성
3. 프로젝트의 결제 활성화
4. API 및 서비스 > API 사용 설정 > "Compute Engine API" 검색 > 사용 설정

---

## 3) VM 생성

**머신 유형:**

| 유형     | 사양                     | 비용               | 비고               |
| -------- | ------------------------ | ------------------ | ------------------ |
| e2-small | 2 vCPU, 2GB RAM          | 월 ~$12            | 권장               |
| e2-micro | 2 vCPU (공유), 1GB RAM   | 무료 티어 대상     | 부하 시 OOM 가능   |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1. Compute Engine > VM 인스턴스 > 인스턴스 만들기로 이동
2. 이름: `openclaw-gateway`
3. 리전: `us-central1`, 영역: `us-central1-a`
4. 머신 유형: `e2-small`
5. 부팅 디스크: Debian 12, 20GB
6. 만들기

---

## 4) VM에 SSH 접속

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:**

Compute Engine 대시보드에서 VM 옆의 "SSH" 버튼을 클릭합니다.

참고: VM 생성 후 SSH 키 전파에 1-2분이 걸릴 수 있습니다. 연결이 거부되면 잠시 기다렸다가 다시 시도하세요.

---

## 5) Docker 설치 (VM에서)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

그룹 변경이 적용되도록 로그아웃 후 다시 로그인:

```bash
exit
```

다시 SSH 접속:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

확인:

```bash
docker --version
docker compose version
```

---

## 6) OpenClaw 저장소 클론

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

이 가이드는 바이너리 영속성을 보장하기 위해 커스텀 이미지를 빌드한다고 가정합니다.

---

## 7) 영구 호스트 디렉토리 생성

Docker 컨테이너는 일시적입니다.
모든 장기 상태는 호스트에 저장되어야 합니다.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

---

## 8) 환경 변수 구성

저장소 루트에 `.env`를 생성합니다.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

강력한 시크릿 생성:

```bash
openssl rand -hex 32
```

**이 파일을 커밋하지 마세요.**

---

## 9) Docker Compose 구성

`docker-compose.yml`을 생성하거나 업데이트합니다.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # 권장: VPS에서 게이트웨이를 loopback으로만 유지; SSH 터널로 접근.
      # 공개적으로 노출하려면 `127.0.0.1:` 접두사를 제거하고 적절히 방화벽을 설정하세요.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

---

## 10) 필요한 바이너리를 이미지에 내장 (중요)

실행 중인 컨테이너 안에 바이너리를 설치하는 것은 함정입니다.
런타임에 설치된 것은 재시작 시 모두 사라집니다.

스킬에 필요한 모든 외부 바이너리는 이미지 빌드 시 설치해야 합니다.

아래 예시는 세 가지 일반적인 바이너리만 보여줍니다:

- Gmail 접근용 `gog`
- Google Places용 `goplaces`
- WhatsApp용 `wacli`

이것은 예시이며 전체 목록이 아닙니다.
같은 패턴을 사용하여 필요한 만큼 바이너리를 설치할 수 있습니다.

나중에 추가 바이너리에 의존하는 새 스킬을 추가하면 다음을 수행해야 합니다:

1. Dockerfile 업데이트
2. 이미지 재빌드
3. 컨테이너 재시작

**Dockerfile 예시**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 예시 바이너리 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 예시 바이너리 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 예시 바이너리 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 같은 패턴으로 아래에 더 많은 바이너리 추가

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

---

## 11) 빌드 및 실행

```bash
docker compose build
docker compose up -d openclaw-gateway
```

바이너리 확인:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

예상 출력:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

---

## 12) 게이트웨이 확인

```bash
docker compose logs -f openclaw-gateway
```

성공:

```
[gateway] listening on ws://0.0.0.0:18789
```

---

## 13) 노트북에서 접근

게이트웨이 포트를 포워딩하는 SSH 터널 생성:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

브라우저에서 열기:

`http://127.0.0.1:18789/`

게이트웨이 토큰을 붙여넣으세요.

---

## 어디에 무엇이 영속화되는지 (기준 문서)

OpenClaw은 Docker에서 실행되지만, Docker가 기준 문서는 아닙니다.
모든 장기 상태는 재시작, 재빌드, 재부팅에서 살아남아야 합니다.

| 구성 요소         | 위치                              | 영속성 메커니즘        | 비고                             |
| ----------------- | --------------------------------- | ---------------------- | -------------------------------- |
| 게이트웨이 설정   | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | `openclaw.json`, 토큰 포함       |
| 모델 인증 프로필  | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | OAuth 토큰, API 키               |
| 스킬 설정         | `/home/node/.openclaw/skills/`    | 호스트 볼륨 마운트     | 스킬 수준 상태                   |
| 에이전트 워크스페이스 | `/home/node/.openclaw/workspace/` | 호스트 볼륨 마운트  | 코드 및 에이전트 아티팩트        |
| WhatsApp 세션     | `/home/node/.openclaw/`           | 호스트 볼륨 마운트     | QR 로그인 유지                   |
| Gmail 키링        | `/home/node/.openclaw/`           | 호스트 볼륨 + 비밀번호 | `GOG_KEYRING_PASSWORD` 필요      |
| 외부 바이너리     | `/usr/local/bin/`                 | Docker 이미지          | 빌드 시 내장 필수                |
| Node 런타임       | 컨테이너 파일시스템               | Docker 이미지          | 이미지 빌드마다 재빌드           |
| OS 패키지         | 컨테이너 파일시스템               | Docker 이미지          | 런타임에 설치하지 마세요         |
| Docker 컨테이너   | 일시적                            | 재시작 가능            | 삭제해도 안전                    |

---

## 업데이트

VM에서 OpenClaw을 업데이트하려면:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

---

## 문제 해결

**SSH 연결 거부**

VM 생성 후 SSH 키 전파에 1-2분이 걸릴 수 있습니다. 잠시 기다렸다가 다시 시도하세요.

**OS Login 문제**

OS Login 프로필을 확인하세요:

```bash
gcloud compute os-login describe-profile
```

계정에 필요한 IAM 권한이 있는지 확인하세요 (Compute OS Login 또는 Compute OS Admin Login).

**메모리 부족 (OOM)**

e2-micro를 사용하면서 OOM이 발생하면, e2-small 또는 e2-medium으로 업그레이드하세요:

```bash
# 먼저 VM 중지
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# 머신 유형 변경
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# VM 시작
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

---

## 서비스 계정 (보안 모범 사례)

개인 사용의 경우 기본 사용자 계정으로 충분합니다.

자동화 또는 CI/CD 파이프라인의 경우, 최소 권한을 가진 전용 서비스 계정을 생성하세요:

1. 서비스 계정 생성:

   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Compute Instance Admin 역할 (또는 더 좁은 커스텀 역할) 부여:

   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

자동화에 Owner 역할을 사용하지 마세요. 최소 권한 원칙을 사용하세요.

IAM 역할 세부 정보는 [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles)을 참조하세요.

---

## 다음 단계

- 메시징 채널 설정: [채널](/channels)
- 로컬 장치를 노드로 페어링: [노드](/nodes)
- 게이트웨이 구성: [게이트웨이 구성](/gateway/configuration)
