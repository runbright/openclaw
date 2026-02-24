---
summary: "윈도우(Windows) WSL2 환경에서의 OpenClaw 설치 및 설정 가이드"
read_when:
  - 윈도우 컴퓨터에서 OpenClaw를 설치하고 싶을 때
  - WSL2 설정 방법이나 포트 포워딩(외부 접속) 방법이 궁금할 때
title: "윈도우 (WSL2)"
---

# 윈도우 가이드 (WSL2)

Windows 환경에서는 **WSL2 (Windows Subsystem for Linux)**를 통해 OpenClaw를 실행하는 것을 강력하게 권장합니다. 윈도우 네이티브 환경보다 라이브러리 호환성이 훨씬 뛰어나며 시스템 안정성도 높습니다.

## WSL2 설치 및 준비

1.  **WSL2 설치**: 파워쉘(PowerShell)을 관리자 권한으로 열고 다음 명령어를 입력하세요.
    ```powershell
    wsl --install
    ```
    (설치 후 컴퓨터를 다시 시작해야 할 수 있습니다.)

2.  **systemd 활성화**: OpenClaw의 백그라운드 서비스를 위해 필수입니다. WSL 터미널(Ubuntu 등)에서 다음 명령어를 실행하세요.
    ```bash
    sudo tee /etc/wsl.conf >/dev/null <<'EOF'
    [boot]
    systemd=true
    EOF
    ```
    이후 윈도우 파워쉘에서 `wsl --shutdown`을 입력해 종료한 뒤 다시 WSL을 실행하세요.

## OpenClaw 설치 (WSL 내부)

WSL 터미널이 열리면 [리눅스 설치 과정](/platforms/linux)과 동일하게 진행합니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 외부 접속 설정 (Port Forwarding)

다른 기기(휴대폰 등)에서 윈도우 내부의 WSL 게이트웨이에 접속하려면 윈도우 포트를 WSL IP로 연결해줘야 합니다.

**관리자 권한 파워쉘에서 실행:**
```powershell
# WSL IP 확인 및 포트 18789 연결
$WslIp = (wsl -- hostname -I).Trim().Split(" ")[0]
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=18789 connectaddress=$WslIp connectport=18789

# 방화벽 허용
New-NetFirewallRule -DisplayName "OpenClaw Gateway" -Direction Inbound -Protocol TCP -LocalPort 18789 -Action Allow
```

## 참고 사항
- 현재 윈도우 전용 동반 앱(메뉴바 앱 등)은 준비 중입니다. 당분간은 WSL 내부의 게이트웨이를 사용해 주세요.
- 윈도우 환경에서 더 쉬운 접근법을 찾으신다면 [Docker 설치 가이드](/install/docker)도 좋은 대안이 될 수 있습니다.

---
설치 중 문제가 발생하면 [문제 해결 가이드](/help/troubleshooting)를 확인하세요.
---
