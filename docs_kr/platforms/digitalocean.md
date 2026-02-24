---
summary: "디지털오션(DigitalOcean)에서 OpenClaw 설치 및 운영 가이드"
read_when:
  - 디지털오션 VPS에서 OpenClaw를 설정하려고 할 때
  - 저렴하고 성능이 보장된 서버를 찾고 있을 때
title: "디지털오션"
---

# 디지털오션 가이드 (DigitalOcean)

디지털오션은 윈도우나 맥과 달리 24시간 안정적으로 실행될 수 있는 서버 환경을 구축하기에 가장 쉽고 대중적인 클라우드 플랫폼입니다.

## 비용 및 사양
- **최저 비용**: 월 **$6** (약 8,000원)
- **사양**: 1 vCPU, 1GB RAM, 25GB SSD
- **특징**: 가입 및 서버 생성이 매우 직관적이고 빠릅니다.

## 설치 및 설정 단계

### 1단계: 서버 생성 (Droplet)
1.  **DigitalOcean**에 로그인합니다.
2.  **Create → Droplets**를 클릭합니다.
3.  **OS**: Ubuntu 24.04 LTS를 선택합니다.
4.  **요금제**: Basic → Regular → **$6/mo** 모델을 선택합니다.
5.  SSH 키를 등록하고 생성을 완료합니다.

### 2단계: 시스템 준비
서버 IP로 SSH 접속 후 다음 명령어를 실행합니다:
```bash
apt update && apt upgrade -y
# Node.js 22 설치
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
```

### 3단계: 가상 메모리(Swap) 설정 (1GB RAM 모델 필수)
램 용량이 부족할 수 있으므로 2GB의 스왑 메모리를 추가합니다:
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### 4단계: OpenClaw 설치 및 설정
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
# 초기 설정 마법사 실행
openclaw onboard --install-daemon
```

## 외부에서 접속하기

게이트웨이는 보안을 위해 로컬에서만 작동하도록 설정되어 있습니다. 외부(내 노트북 등)에서 대시보드에 접속하려면 다음 두 가지 방법 중 하나를 선택하세요:

- **방법 A: SSH 터널링 (간편함)**: 내 노트북 터미널에서 실행
  ```bash
  ssh -L 18789:localhost:18789 root@서버IP
  ```
  이후 브라우저에서 `http://localhost:18789` 접속.

- **방법 B: Tailscale (강력 추천)**: 서버와 내 노트북에 Tailscale을 설치하여 VPN으로 연결.

---
무료 서버를 원하신다면 [오라클 클라우드 가이드](/platforms/oracle)를 확인해 보세요.
---
