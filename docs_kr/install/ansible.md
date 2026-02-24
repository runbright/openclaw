---
summary: "Ansible, Tailscale VPN, 방화벽 격리를 활용한 자동화된 보안 강화 OpenClaw 설치"
read_when:
  - 보안 강화된 자동 서버 배포를 원할 때
  - VPN 접근이 가능한 방화벽 격리 설정이 필요할 때
  - 원격 Debian/Ubuntu 서버에 배포할 때
title: "Ansible"
---

# Ansible 설치

OpenClaw을 프로덕션 서버에 배포하는 권장 방법은 **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**을 사용하는 것입니다. 보안 우선 아키텍처를 갖춘 자동화된 설치 도구입니다.

## 빠른 시작

한 줄 명령으로 설치:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 전체 가이드: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> openclaw-ansible 저장소가 Ansible 배포의 기준 문서입니다. 이 페이지는 간략한 개요입니다.

## 제공되는 기능

- 🔒 **방화벽 우선 보안**: UFW + Docker 격리 (SSH + Tailscale만 접근 가능)
- 🔐 **Tailscale VPN**: 서비스를 공개적으로 노출하지 않고 안전한 원격 접근
- 🐳 **Docker**: 격리된 샌드박스 컨테이너, localhost 전용 바인딩
- 🛡️ **심층 방어**: 4계층 보안 아키텍처
- 🚀 **한 줄 명령 설정**: 몇 분 만에 전체 배포 완료
- 🔧 **Systemd 통합**: 보안 강화된 부팅 시 자동 시작

## 요구 사항

- **OS**: Debian 11+ 또는 Ubuntu 20.04+
- **접근 권한**: Root 또는 sudo 권한
- **네트워크**: 패키지 설치를 위한 인터넷 연결
- **Ansible**: 2.14+ (빠른 시작 스크립트가 자동으로 설치)

## 설치되는 항목

Ansible 플레이북이 설치 및 구성하는 항목:

1. **Tailscale** (안전한 원격 접근을 위한 메시 VPN)
2. **UFW 방화벽** (SSH + Tailscale 포트만 허용)
3. **Docker CE + Compose V2** (에이전트 샌드박스용)
4. **Node.js 22.x + pnpm** (런타임 의존성)
5. **OpenClaw** (호스트 기반, 컨테이너화되지 않음)
6. **Systemd 서비스** (보안 강화된 자동 시작)

참고: 게이트웨이는 **호스트에서 직접** 실행됩니다 (Docker 내부가 아님). 하지만 에이전트 샌드박스는 격리를 위해 Docker를 사용합니다. 자세한 내용은 [샌드박싱](/gateway/sandboxing)을 참조하세요.

## 설치 후 설정

설치가 완료되면 openclaw 사용자로 전환하세요:

```bash
sudo -i -u openclaw
```

설치 후 스크립트가 다음 과정을 안내합니다:

1. **온보딩 마법사**: OpenClaw 설정 구성
2. **프로바이더 로그인**: WhatsApp/Telegram/Discord/Signal 연결
3. **게이트웨이 테스트**: 설치 확인
4. **Tailscale 설정**: VPN 메시에 연결

### 빠른 명령어

```bash
# 서비스 상태 확인
sudo systemctl status openclaw

# 실시간 로그 보기
sudo journalctl -u openclaw -f

# 게이트웨이 재시작
sudo systemctl restart openclaw

# 프로바이더 로그인 (openclaw 사용자로 실행)
sudo -i -u openclaw
openclaw channels login
```

## 보안 아키텍처

### 4계층 방어

1. **방화벽 (UFW)**: SSH (22) + Tailscale (41641/udp)만 공개적으로 노출
2. **VPN (Tailscale)**: VPN 메시를 통해서만 게이트웨이에 접근 가능
3. **Docker 격리**: DOCKER-USER iptables 체인이 외부 포트 노출 차단
4. **Systemd 강화**: NoNewPrivileges, PrivateTmp, 비특권 사용자

### 검증

외부 공격 표면 테스트:

```bash
nmap -p- YOUR_SERVER_IP
```

**포트 22** (SSH)만 열려 있어야 합니다. 다른 모든 서비스 (게이트웨이, Docker)는 차단됩니다.

### Docker 가용성

Docker는 게이트웨이 자체를 실행하기 위한 것이 아니라 **에이전트 샌드박스** (격리된 도구 실행)용으로 설치됩니다. 게이트웨이는 localhost에만 바인딩되며 Tailscale VPN을 통해 접근할 수 있습니다.

자세한 샌드박스 구성은 [멀티 에이전트 샌드박스 & 도구](/tools/multi-agent-sandbox-tools)를 참조하세요.

## 수동 설치

자동화보다 수동 제어를 선호하는 경우:

```bash
# 1. 사전 요구 사항 설치
sudo apt update && sudo apt install -y ansible git

# 2. 저장소 클론
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Ansible 컬렉션 설치
ansible-galaxy collection install -r requirements.yml

# 4. 플레이북 실행
./run-playbook.sh

# 또는 직접 실행 (이후 /tmp/openclaw-setup.sh를 수동으로 실행)
# ansible-playbook playbook.yml --ask-become-pass
```

## OpenClaw 업데이트

Ansible 설치 프로그램은 수동 업데이트용으로 OpenClaw을 설정합니다. 표준 업데이트 과정은 [업데이트](/install/updating)를 참조하세요.

Ansible 플레이북을 다시 실행하려면 (예: 구성 변경 시):

```bash
cd openclaw-ansible
./run-playbook.sh
```

참고: 이 작업은 멱등적이며 여러 번 실행해도 안전합니다.

## 문제 해결

### 방화벽이 연결을 차단함

접근이 차단된 경우:

- 먼저 Tailscale VPN을 통해 접근할 수 있는지 확인하세요
- SSH 접근 (포트 22)은 항상 허용됩니다
- 게이트웨이는 설계상 Tailscale을 통해서**만** 접근 가능합니다

### 서비스가 시작되지 않음

```bash
# 로그 확인
sudo journalctl -u openclaw -n 100

# 권한 확인
sudo ls -la /opt/openclaw

# 수동 시작 테스트
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker 샌드박스 문제

```bash
# Docker 실행 중인지 확인
sudo systemctl status docker

# 샌드박스 이미지 확인
sudo docker images | grep openclaw-sandbox

# 샌드박스 이미지가 없으면 빌드
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### 프로바이더 로그인 실패

`openclaw` 사용자로 실행하고 있는지 확인하세요:

```bash
sudo -i -u openclaw
openclaw channels login
```

## 고급 구성

자세한 보안 아키텍처 및 문제 해결:

- [보안 아키텍처](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [기술 세부 사항](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [문제 해결 가이드](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## 관련 문서

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 전체 배포 가이드
- [Docker](/install/docker) — 컨테이너화된 게이트웨이 설정
- [샌드박싱](/gateway/sandboxing) — 에이전트 샌드박스 구성
- [멀티 에이전트 샌드박스 & 도구](/tools/multi-agent-sandbox-tools) — 에이전트별 격리
