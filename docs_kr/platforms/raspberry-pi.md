---
summary: "라즈베리 파이(Raspberry Pi)에서 OpenClaw 실행하기 (저비용 셀프 호스팅 설정)"
read_when:
  - 라즈베리 파이에 OpenClaw를 설치하려고 할 때
  - ARM 기기에서 OpenClaw를 실행할 때
  - 저렴한 비용으로 24시간 가동되는 개인용 AI를 만들고 싶을 때
title: "라즈베리 파이"
---

# 라즈베리 파이 가이드 (Raspberry Pi)

라즈베리 파이는 저렴한 비용으로 24시간 중단 없이 작동하는 개인용 AI 비서를 만들기에 가장 완벽한 기기입니다. 월간 구독료 없이 나만의 게이트웨이를 구축해 보세요.

## 하드웨어 권장 사양

- **라즈베리 파이 5**: 가장 빠르고 쾌적합니다 (강력 추천).
- **라즈베리 파이 4**: 2GB 이상의 RAM 모델이라면 충분히 잘 작동합니다.
- **OS**: 반드시 **Raspberry Pi OS Lite (64-bit)**를 사용하세요. (데스크탑 화면이 없는 서버용 버전)

## 설치 단계

### 1. OS 설치 및 설정
[Raspberry Pi Imager](https://www.raspberrypi.com/software/)를 사용하여 SD 카드에 OS를 작성합니다. 설정(톱니바퀴 아이콘)에서 SSH를 활성화하고 사용자 계정을 미리 만들어두면 편리합니다.

### 2. 필수 패키지 설치
SSH로 접속한 뒤 시스템을 업데이트하고 필요한 도구들을 설치합니다.
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl build-essential
```

### 3. Node.js 22 설치
OpenClaw 실행을 위해 최신 Node.js가 필요합니다.
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

### 4. 스왑(Swap) 설정 (매우 중요)
RAM이 부족하여 시스템이 멈추는 것을 방지하기 위해 가상 메모리(Swap)를 설정합니다 (최소 2GB 권장).
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 5. OpenClaw 설치
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 최적화 팁

- **USB SSD 사용**: SD 카드보다 속도가 훨씬 빠르고 수명이 깁니다.
- **클라우드 모델 활용**: 라즈베리 파이 기기 자체에서 AI 모델(LLM)을 직접 돌리는 것은 매우 느립니다. 모델은 GPT-4나 Claude 같은 클라우드 API를 사용하고, 파이는 이를 제어하는 게이트웨이 역할만 수행하게 하세요.
- **원격 접속**: 외부에서 파이에 접속하고 싶다면 **Tailscale**을 설치하는 것이 가장 쉽고 안전합니다.

## 문제 해결

- **메모리 부족**: `free -h` 명령어로 메모리 사용량을 확인하고, 불필요한 서비스(`bluetooth` 등)를 끕니다.
- **발열**: 파이 4나 5는 발열이 있을 수 있으니 방열판이나 쿨링 팬 사용을 권장합니다.

---
더 구체적인 리눅스 설정은 [리눅스 가이드](/platforms/linux)를 참고하세요.
---
