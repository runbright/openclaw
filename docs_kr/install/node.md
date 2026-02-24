---
title: "Node.js 설정"
summary: "OpenClaw 실행을 위한 Node.js 설치 및 설정 가이드 — 버전 요구 사항, 설치 방법 및 PATH 문제 해결"
read_when:
  - OpenClaw 설치 전 Node.js를 먼저 설치해야 할 때
  - OpenClaw를 설치했지만 명령어를 찾을 수 없다는 오류가 발생할 때
---

# Node.js 설정

OpenClaw는 **Node.js 22 버전 이상**이 필요합니다. [설치 스크립트](/install#install-methods)를 사용하면 Node.js를 자동으로 감지하고 설치하지만, 직접 설치하거나 설정하고 싶다면 이 가이드를 참고하세요.

## 버전 확인

터미널에서 현재 설치된 버전을 확인해 보세요:

```bash
node -v
```

결과가 `v22.x.x` 이상이면 준비 완료입니다. 만약 설치되어 있지 않거나 버전이 낮다면 아래 방법 중 하나로 설치하세요.

## Node.js 설치 방법

<Tabs>
  <Tab title="macOS">
    **Homebrew 사용 (권장):**
    ```bash
    brew install node
    ```
  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian:**
    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```
  </Tab>
  <Tab title="Windows">
    **winget 사용 (권장):**
    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```
  </Tab>
</Tabs>

## 문제 해결 (Troubleshooting)

### `openclaw: command not found` 오류

이 오류는 대부분 npm의 전역 바이너리 경로가 시스템 `PATH`에 추가되지 않았을 때 발생합니다.

1.  **경로 확인**: `npm prefix -g` 명령어를 실행하여 npm 전역 경로를 알아냅니다.
2.  **PATH 추가**:
    - **macOS / Linux**: `~/.zshrc` 또는 `~/.bashrc` 파일 끝에 아래 내용을 추가합니다.
      ```bash
      export PATH="$(npm prefix -g)/bin:$PATH"
      ```
    - **Windows**: 제어판 '시스템 환경 변수 편집'에서 'Path' 항목에 위에서 확인한 경로를 추가합니다.

설정 후에는 터미널을 완전히 껐다가 다시 실행하여 변경 사항을 적용하세요.

---
성공적으로 설치했다면 [설치 가이드](/install)로 돌아가 OpenClaw 설치를 계속 진행하세요.
---
