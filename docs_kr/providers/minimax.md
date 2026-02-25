---
summary: "OpenClaw에서 MiniMax M2.1 모델을 사용하는 방법"
read_when:
  - MiniMax 모델을 OpenClaw에 연동하고 싶을 때
  - MiniMax 코딩 플랜(Coding Plan) 또는 API 키 설정을 확인해야 할 때
title: "MiniMax"
---

# MiniMax

MiniMax는 고성능 AI 모델인 **M2/M2.1** 시리즈를 개발한 회사입니다. 특히 최신 모델인 **MiniMax M2.1**은 복잡한 실제 작업을 처리하기 위해 최적화되었으며, 강력한 코딩 능력을 자랑합니다.

## 주요 특징 (M2.1)
- **강력한 다국어 코딩**: Rust, Java, Go, C++, JS/TS 등 다양한 언어 지원
- **웹/앱 개발 최적화**: 미적 완성도가 높은 출력물 생성
- **에이전트 호환성**: 에이전트 도구 및 컨텍스트 관리 능력이 우수함
- **효율성**: 토큰 사용량이 적고 빠른 응답 속도 제공

## 설정 방법 선택

### 1. MiniMax OAuth (코딩 플랜) — 권장
API 키 없이 OAuth 인증을 통해 간편하게 설정할 수 있습니다.
```bash
# 플러그인 활성화 및 자동 설정 시작
openclaw onboard --auth-choice minimax-portal
```

### 2. MiniMax M2.1 (API 키 입력 형식)
이미 API 키를 보유하고 있는 경우 Anthropic 호환 엔드포인트를 통해 설정합니다.
- `openclaw configure` 실행 후 **Model/auth** -> **MiniMax M2.1** 선택

## 설정 예시 (`openclaw.json`)
```json5
{
  "env": { "MINIMAX_API_KEY": "sk-..." },
  "agents": { "defaults": { "model": { "primary": "minimax/MiniMax-M2.1" } } },
  "models": {
    "providers": {
      "minimax": {
        "baseUrl": "https://api.minimax.io/anthropic",
        "apiKey": "${MINIMAX_API_KEY}",
        "api": "anthropic-messages"
      }
    }
  }
}
```

## 참고
- **모델 식별자**: 대소문자를 구분합니다. (`minimax/MiniMax-M2.1`)
- **버전**: `MiniMax-M2.1-lightning`은 속도에 최적화된 빠른 버전입니다.
- **지역**: 글로벌 사용자는 `api.minimax.io`, 중국 내 사용자는 `api.minimaxi.com` 엔드포인트를 사용합니다.

---
문제 발생 시 `openclaw models list` 명령어로 모델이 정상적으로 로드되었는지 확인하세요.
---
