---
summary: "리눅스(Linux) 지원 및 백그라운드 서비스 설정 가이드"
read_when:
  - 리눅스 서버나 데스크탑에서 OpenClaw를 설정하고 싶을 때
  - 리눅스용 백그라운드 서비스(systemd) 명령어를 알고 싶을 때
title: "리눅스 가이드"
---

# 리눅스 가이드 (Linux)

Linux에서는 OpenClaw 게이트웨이가 완벽하게 지원됩니다. 서버용(Headless) 환경에서 24시간 작동시키기에 최적의 운영체제입니다.

## 빠른 설치 단계 (VPS/서버)

1.  **Node 22 이상 설치**: [Node.js 설정 가이드](/install/node)를 참고하세요.
2.  **프로그램 설치**:
    ```bash
    npm i -g openclaw@latest
    ```
3.  **초기 설정 및 서비스 등록**:
    ```bash
    openclaw onboard --install-daemon
    ```

## 백그라운드 서비스 관리 (systemd)

OpenClaw는 리눅스에서 `systemd` 사용자 서비스를 사용합니다.

- **서비스 상태 확인**:
  ```bash
  systemctl --user status openclaw-gateway.service
  ```
- **서비스 재시작**:
  ```bash
  systemctl --user restart openclaw-gateway.service
  ```
- **로그 실시간 확인**:
  ```bash
  journalctl --user -u openclaw-gateway.service -f
  ```

## 라즈베리 파이 (Raspberry Pi)
라즈베리 파이 4 이상의 모델에서 OpenClaw를 원활하게 실행할 수 있습니다. 자세한 내용은 [라즈베리 파이 전용 가이드](/platforms/raspberry-pi)를 확인하세요.

---
추가적인 정보는 [게이트웨이 운영 가이드](/gateway)를 참고하세요.
---
