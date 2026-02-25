---
summary: "OpenClaw에서 지원하는 모델 제공업체(LLM) 요약 및 빠른 시작"
read_when:
  - 모델 제공업체를 빠르게 선택하고 싶을 때
  - LLM 인증 및 모델 선택 예시를 확인하고 싶을 때
title: "모델 제공업체 시작하기"
---

# 모델 제공업체 (Model Providers)

OpenClaw는 다양한 LLM 제공업체를 지원합니다. 하나를 선택해 인증을 마친 후, 기본 모델을 `제공업체/모델명` 형식으로 지정하여 시작하세요.

## 주요 추천: 베니스 (Venice AI)
개인정보 보호를 중시하면서도 강력한 성능이 필요하다면 베니스(Venice AI)를 추천합니다.
- 기본값: `venice/llama-3.3-70b`
- 최고 성능: `venice/claude-opus-45`

상세 가이드는 [베니스 가이드](/providers/venice)를 참고하세요.

## 빠른 시작 (2단계)
1.  **인증**: `openclaw onboard`를 실행하여 제공업체 인증을 진행합니다.
2.  **기본 모델 설정**: `openclaw.json` 파일에 기본 모델을 지정합니다.
    ```json5
    {
      "agents": { "defaults": { "model": { "primary": "anthropic/claude-opus-4-6" } } }
    }
    ```

## 지원 제공업체 (목록)
- [Anthropic (Claude)](/providers/anthropic)
- [OpenAI (GPT-4o)](/providers/openai)
- [OpenRouter](/providers/openrouter)
- [Mistral](/providers/mistral)
- [MiniMax](/providers/minimax)
- [Moonshot AI (Kimi)](/providers/moonshot)
- [Amazon Bedrock](/providers/bedrock)
- 그 외 다수 (GLM, Z.AI, Qianfan 등)

---
더 상세한 카탈로그와 고급 설정은 [모델 제공업체 개념 설명](/concepts/model-providers)을 확인하세요.
---
