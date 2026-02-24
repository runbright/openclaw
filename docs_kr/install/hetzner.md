---
summary: "저렴한 Hetzner VPS (Docker)에서 영구 상태와 내장 바이너리를 갖춘 OpenClaw 게이트웨이를 24/7 실행"
read_when:
  - 클라우드 VPS에서 (노트북이 아닌) OpenClaw을 24/7 실행하고 싶을 때
  - 자체 VPS에서 프로덕션급 상시 가동 게이트웨이를 원할 때
  - 영속성, 바이너리, 재시작 동작에 대한 완전한 제어를 원할 때
  - Hetzner 또는 유사 프로바이더에서 Docker로 OpenClaw을 실행할 때
title: "Hetzner"
---

# Hetzner에서 OpenClaw 실행 (Docker, 프로덕션 VPS 가이드)

## 목표

Hetzner VPS에서 Docker를 사용하여 영구 상태, 내장 바이너리, 안전한 재시작 동작을 갖춘 지속적인 OpenClaw 게이트웨이를 실행합니다.

"~$5에 OpenClaw 24/7"을 원한다면, 이것이 가장 간단하고 신뢰할 수 있는 설정입니다.
Hetzner 가격은 변경될 수 있습니다; 가장 작은 Debian/Ubuntu VPS를 선택하고 OOM이 발생하면 확장하세요.

보안 모델 참고:

- 모든 사람이 같은 신뢰 경계 안에 있고 런타임이 업무 전용인 경우 회사 공유 에이전트는 괜찮습니다.
- 엄격한 분리를 유지하세요: 전용 VPS/런타임 + 전용 계정; 해당 호스트에 개인 Apple/Google/브라우저/비밀번호 관리자 프로필을 두지 마세요.
- 사용자가 서로에게 적대적인 경우, 게이트웨이/호스트/OS 사용자별로 분리하세요.

[보안](/gateway/security) 및 [VPS 호스팅](/vps)을 참조하세요.

## 무엇을 하는 건가요 (쉽게 설명)?

- 작은 Linux 서버 임대 (Hetzner VPS)
- Docker 설치 (격리된 앱 런타임)
- Docker에서 OpenClaw 게이트웨이 시작
- 호스트에 `~/.openclaw` + `~/.openclaw/workspace` 영속화 (재시작/재빌드에도 유지)
- SSH 터널을 통해 노트북에서 Control UI 접근

게이트웨이는 다음을 통해 접근할 수 있습니다:

- 노트북에서 SSH 포트 포워딩
- 방화벽과 토큰을 직접 관리하는 경우 직접 포트 노출

이 가이드는 Hetzner의 Ubuntu 또는 Debian을 기준으로 합니다.
다른 Linux VPS를 사용하는 경우 패키지를 적절히 매핑하세요.
일반적인 Docker 흐름은 [Docker](/install/docker)를 참조하세요.

---

## 빠른 경로 (경험 있는 운영자)

1. Hetzner VPS 프로비저닝
2. Docker 설치
3. OpenClaw 저장소 클론
4. 영구 호스트 디렉토리 생성
5. `.env` 및 `docker-compose.yml` 구성
6. 필요한 바이너리를 이미지에 내장
7. `docker compose up -d`
8. 영속성 및 게이트웨이 접근 확인

---

## 필요한 것

- root 접근이 가능한 Hetzner VPS
- 노트북에서 SSH 접근
- SSH + 복사/붙여넣기에 대한 기본적인 익숙함
- 약 20분
- Docker 및 Docker Compose
- 모델 인증 자격 증명
- 선택적 프로바이더 자격 증명
  - WhatsApp QR
  - Telegram 봇 토큰
  - Gmail OAuth

---

## 1) VPS 프로비저닝

Hetzner에서 Ubuntu 또는 Debian VPS를 생성합니다.

root로 연결:

```bash
ssh root@YOUR_VPS_IP
```

이 가이드는 VPS가 상태 유지형임을 가정합니다.
일회용 인프라로 취급하지 마세요.

---

## 2) Docker 설치 (VPS에서)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

확인:

```bash
docker --version
docker compose version
```

---

## 3) OpenClaw 저장소 클론

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

이 가이드는 바이너리 영속성을 보장하기 위해 커스텀 이미지를 빌드한다고 가정합니다.

---

## 4) 영구 호스트 디렉토리 생성

Docker 컨테이너는 일시적입니다.
모든 장기 상태는 호스트에 저장되어야 합니다.

```bash
mkdir -p /root/.openclaw/workspace

# 컨테이너 사용자 (uid 1000)에 소유권 설정:
chown -R 1000:1000 /root/.openclaw
```

---

## 5) 환경 변수 구성

저장소 루트에 `.env`를 생성합니다.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

강력한 시크릿 생성:

```bash
openssl rand -hex 32
```

**이 파일을 커밋하지 마세요.**

---

## 6) Docker Compose 구성

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
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured`는 초기 부트스트랩 편의를 위한 것이며, 적절한 게이트웨이 구성을 대체하지 않습니다. 여전히 인증 (`gateway.auth.token` 또는 비밀번호)을 설정하고 배포에 안전한 바인드 설정을 사용하세요.

---

## 7) 필요한 바이너리를 이미지에 내장 (중요)

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

## 8) 빌드 및 실행

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

## 9) 게이트웨이 확인

```bash
docker compose logs -f openclaw-gateway
```

성공:

```
[gateway] listening on ws://0.0.0.0:18789
```

노트북에서:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

열기:

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

## Infrastructure as Code (Terraform)

인프라를 코드로 관리하는 워크플로우를 선호하는 팀을 위해, 커뮤니티에서 유지 관리하는 Terraform 설정이 다음을 제공합니다:

- 원격 상태 관리를 갖춘 모듈러 Terraform 구성
- cloud-init을 통한 자동 프로비저닝
- 배포 스크립트 (부트스트랩, 배포, 백업/복원)
- 보안 강화 (방화벽, UFW, SSH 전용 접근)
- 게이트웨이 접근을 위한 SSH 터널 구성

**저장소:**

- 인프라: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
- Docker 구성: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

이 접근 방식은 위의 Docker 설정을 재현 가능한 배포, 버전 관리된 인프라, 자동화된 재해 복구로 보완합니다.

> **참고:** 커뮤니티에서 유지 관리합니다. 문제 또는 기여는 위의 저장소 링크를 참조하세요.
