---
summary: "통합 모델 접속 및 비용 추적을 위해 LiteLLM 프록시를 사용하는 방법"
read_when:
  - OpenClaw를 LiteLLM 프록시를 통해 운영하고 싶을 때
  - 통합 비용 추적, 로깅 또는 모델 라우팅 기능이 필요할 때
title: "LiteLLM"
---

# LiteLLM

[LiteLLM](https://litellm.ai)은 100개 이상의 모델 제공업체를 하나의 통합 API로 연결해주는 오픈 소스 LLM 게이트웨이입니다. OpenClaw를 LiteLLM을 통해 실행하면 중앙 완성형 비용 추적, 로깅, 그리고 OpenClaw 설정을 바꾸지 않고도 백엔드 모델을 자유롭게 교체할 수 있는 유연성을 얻을 수 있습니다.

## OpenClaw와 LiteLLM을 함께 사용하는 이유
- **비용 추적**: 모든 모델에서 OpenClaw가 사용하는 정확한 비용을 한눈에 확인합니다.
- **유연한 라우팅**: 설정 변경 없이 Claude, GPT-4, Gemini 등 제공업체를 자유롭게 오갈 수 있습니다.
- **가상 키 관리**: OpenClaw 전용 API 키를 생성하고 월별 사용 한도를 설정할 수 있습니다.
- **로깅 및 디버깅**: 모든 요청과 응답 로그를 남겨 문제 해결 시 활용합니다.
- **장애 복구(Failover)**: 기본 제공업체에 문제가 생겼을 때 자동으로 다른 모델로 전환됩니다.

## 빠른 설정 방법
```bash
# 온보딩 시 선택
openclaw onboard --auth-choice litellm-api-key
```

## 설정 예시 (`openclaw.json`)
```json5
{
  "models": {
    "providers": {
      "litellm": {
        "baseUrl": "http://localhost:4000", // LiteLLM 프록시 주소
        "apiKey": "${LITELLM_API_KEY}",
        "api": "openai-completions",
        "models": [
          {
            "id": "claude-opus-4-6",
            "name": "Claude Opus 4.6"
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": { "primary": "litellm/claude-opus-4-6" }
    }
  }
}
```

## 참고
- **기본 포트**: LiteLLM은 기본적으로 `http://localhost:4000`에서 실행됩니다.
- **호환성**: OpenClaw의 모든 기능은 LiteLLM 프록시를 통해서도 제한 없이 작동합니다.
- **가상 키 생성**: 특정 예산 범위 내에서만 작동하는 전용 키를 생성하여 보안을 높일 수 있습니다.

---
더 구체적인 사용법은 [LiteLLM 공식 문서](https://docs.litellm.ai)를 참고하세요.
---
