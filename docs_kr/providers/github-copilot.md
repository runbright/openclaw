---
summary: "디바이스 인증(Device Flow)을 통해 OpenClaw에서 깃허브 코파일럿(GitHub Copilot)을 사용하는 방법"
read_when:
  - 깃허브 코파일럿을 모델 제공업체로 사용하고 싶을 때
  - `openclaw models auth login-github-copilot` 로그인 과정을 확인해야 할 때
title: "깃허브 코파일럿 (GitHub Copilot)"
---

# 깃허브 코파일럿 (GitHub Copilot)

깃허브 코파일럿(GitHub Copilot)은 깃허브의 AI 코딩 어시스턴트입니다. OpenClaw는 코파일럿의 모델들을 직접 가져와 에이전트의 두뇌로 사용할 수 있도록 두 가지 방식을 지원합니다.

## OpenClaw에서 코파일럿을 사용하는 두 가지 방식

### 1) 내장 깃허브 코파일럿 제공업체 (`github-copilot`) — 권장
별도의 VS Code 확장 프로그램 없이, 터미널에서 직접 깃허브 계정으로 로그인하여 토큰을 얻는 방식입니다. 가장 간단하고 권장되는 방법입니다.

### 2) 코파일럿 프록시 플러그인 (`copilot-proxy`)
VS Code에 설치된 **Copilot Proxy** 확장을 로컬 브리지로 사용하는 방식입니다. VS Code 환경에서 이미 코파일럿을 사용 중인 경우 유용합니다.

## 빠른 설정 방법 (내장 방식)

1.  **로그인 실행**: 터미널에서 아래 명령어를 입력합니다.
    ```bash
    openclaw models auth login-github-copilot
    ```
2.  **인증**: 터미널에 나타나는 URL에 접속하여 일회용 코드를 입력하고 권한을 승인합니다.
3.  **기본 모델 설정**:
    ```bash
    openclaw models set github-copilot/gpt-4o
    ```

## 설정 예시 (`openclaw.json`)
```json5
{
  "agents": {
    "defaults": {
      "model": { "primary": "github-copilot/gpt-4o" }
    }
  }
}
```

## 참고 사항
- **권한**: 사용 중인 깃허브 플랜(Plan)에 따라 사용 가능한 모델이 다를 수 있습니다.
- **인증 보안**: 로그인을 통해 얻은 토큰은 안전하게 저장되며, OpenClaw 실행 시 자동으로 코파일럿 API 토큰으로 교환됩니다.
- **오류 해결**: 특정 모델이 거부될 경우 다른 모델 ID(예: `github-copilot/gpt-4.1`)를 시도해 보세요.

---
다른 모델 제공업체에 대한 정보는 [모델 제공업체 개요](/providers/index)를 확인하세요.
---
