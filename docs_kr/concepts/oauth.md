---
summary: "OpenClaw의 OAuth 인증: 토큰 교환, 저장 방식 및 멀티 계정 관리 패턴 가이드"
read_when:
  - OpenClaw의 OAuth 작동 방식을 이해하고 싶을 때
  - 토큰 만료나 로그아웃 문제를 해결할 때
  - 여러 계정 또는 프로필 라우팅 설정을 원할 때
title: "OAuth"
---

# OAuth 및 구독 인증

OpenClaw는 OpenAI Codex(ChatGPT)나 Anthropic(Claude)과 같은 서비스의 유료 구독 정보를 활용할 수 있도록 OAuth인증 및 설정 토큰(Setup-token) 방식을 지원합니다.

## 인증 라이브러리 (Auth Storage)

OpenClaw는 `auth-profiles.json` 파일을 사용하여 인증 정보를 관리합니다. 이 파일에는 사용자의 API 키와 OAuth 토큰이 함께 저장됩니다.

- **저장 위치**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **보안**: 토큰 정보는 에이전트별로 격리된 공간에 저장됩니다.

## 주요 연동 방식

### 1. Anthropic (Claude Pro/Max) 설정 토큰
Anthropic 구독 사용자는 앤스로픽 CLI를 통해 발급받은 설정 토큰을 OpenClaw에 붙여넣어 연동할 수 있습니다.

```bash
# Claude CLI에서 'claude setup-token' 실행 후 토큰 복사
openclaw models auth setup-token --provider anthropic
```

### 2. OpenAI Codex (ChatGPT OAuth)
ChatGPT 유료 구독 정보를 사용하기 위해 브라우저를 통한 로그인 과정을 거칩니다.

```bash
openclaw models auth login --provider openai-codex
```
- 실행 시 브라우저가 열리고, 로그인을 완료하면 자동으로 토큰이 OpenClaw로 전달됩니다. 만약 자동 전달이 안 되는 경우 터미널에 리다이렉트 URL을 붙여넣어 수동으로 완료할 수 있습니다.

## 토큰 갱신 (Refresh)

OAuth 토큰은 만료 시간이 있습니다. OpenClaw는 백그라운드에서 자동으로 토큰 만료 여부를 확인하고, 필요할 때 리프레시 토큰(Refresh token)을 사용해 새로운 액세스 토큰을 발급받습니다. 사용자가 수동으로 로그인 과정을 반복할 필요는 없습니다.

## 멀티 계정 관리

여러 개의 계정(예: 개인용, 업무용)을 사용하는 경우 두 가지 방법이 있습니다:

1. **분리된 에이전트 (추천)**: 각 계정마다 별도의 에이전트 환경을 구성합니다.
   ```bash
   openclaw agents add personal
   openclaw agents add work
   ```
2. **하나의 에이전트 내 멀티 프로필**: `auth-profiles.json`에 여러 프로필을 저장하고 `/model` 명령어에 `@프로필ID`를 붙여 전환합니다.
   - 예: `/model Opus@anthropic:work`

상세한 페일오버(장애 전환) 규칙이나 모델 선택 로직은 [모델 페일오버 문서](/concepts/model-failover)를 참고하십시오.
---
