---
summary: "OpenClaw에서 OpenRouter의 통합 API로 다양한 모델 사용하기"
read_when:
  - 여러 LLM을 위한 단일 API 키를 원할 때
  - OpenClaw에서 OpenRouter를 통해 모델을 실행하고 싶을 때
title: "OpenRouter"
---

# OpenRouter

OpenRouter는 단일 엔드포인트와 API 키로 다양한 모델에 요청을 라우팅하는 **통합 API**를 제공합니다. OpenAI와 호환되므로 대부분의 OpenAI SDK는 기본 URL만 변경하면 작동합니다.

## CLI 설정

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 설정 스니펫

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## 참고 사항

- 모델 참조는 `openrouter/<provider>/<model>` 형식입니다.
- 더 많은 모델/프로바이더 옵션은 [/concepts/model-providers](/concepts/model-providers)를 참조하세요.
- OpenRouter는 내부적으로 API 키를 사용하여 Bearer 토큰 인증을 수행합니다.
