---
summary: "오라클 클라우드(Oracle Cloud) Always Free ARM 서버에서 OpenClaw 실행하기"
read_when:
  - 오라클 클라우드에서 OpenClaw를 설정하려고 할 때
  - 무료 VPS 호스팅을 찾고 있을 때
  - 24시간 가동되는 서버 구축 방법이 궁금할 때
title: "오라클 클라우드"
---

# 오라클 클라우드 가이드 (Oracle Cloud)

오라클 클라우드(OCI)의 **Always Free** 티어를 사용하면 비용 부담 없이 24시간 작동하는 AI 게이트웨이를 구축할 수 있습니다. 

## 하드웨어 사양 (무료 티어)
- **CPU**: Ampere ARM (최대 4 OCPU)
- **RAM**: 최대 24GB
- **디스크**: 최대 200GB (무료 용량 범위 내)
- **특징**: 무료 사양치고 매우 넉넉하지만, 가입 절차가 까다롭고 빈 자리가 없을 때가 많습니다.

## 설치 및 설정 단계

### 1단계: 인스턴스 생성
1.  오라클 클라우드 콘솔에서 **인스턴스 생성**을 선택합니다.
2.  **OS**: Ubuntu 24.04 (aarch64)를 선택합니다.
3.  **구성(Shape)**: `VM.Standard.A1.Flex` (Ampere)를 선택합니다.
4.  사용자의 SSH 공개키를 추가하고 생성을 완료합니다.

### 2단계: 시스템 준비
생성된 인스턴스에 SSH로 접속한 뒤 기본 환경을 구축합니다.
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
# 사용자 서비스 지속 실행 설정 (로그아웃 해도 유지)
sudo loginctl enable-linger ubuntu
```

### 3단계: Tailscale 설치 (보안 접속)
공용 인터넷에 포트를 개방하는 대신 Tailscale을 사용해 안전하게 접속하는 것을 권장합니다.
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

### 4단계: OpenClaw 설치
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```
설치 중 마법사가 뜨면 "나중에 하기(Do this later)"를 선택하고 아래 설정을 진행하세요.

### 5단계: 게이트웨이 보안 설정
```bash
# 로컬에서만 듣도록 설정
openclaw config set gateway.bind loopback
# 인증 토큰 생성
openclaw doctor --generate-gateway-token
# Tailscale을 통해 외부 노출 (HTTPS 지원)
openclaw config set gateway.tailscale.mode serve
systemctl --user restart openclaw-gateway
```

## 대시보드 접속
Tailscale이 연결된 기기에서 다음 주소로 접속하세요:
`https://openclaw.<내테일넷이름>.ts.net/`

## 주의 사항 (Troubleshooting)
- **용량 부족 (Out of capacity)**: 무료 티어는 인기가 많아 자리가 없을 수 있습니다. 다른 지역(Region)을 선택하거나 나중에 다시 시도해 보세요.
- **ARM 호환성**: 대부분의 도구는 잘 작동하지만, 일부 오래된 도구는 ARM 환경을 지원하지 않을 수 있습니다.

---
유료지만 더 쉬운 가입을 원하신다면 [DigitalOcean 가이드](/platforms/digitalocean)를 확인하세요.
---
