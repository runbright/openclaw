---
summary: "OpenClaw에서 지원하는 모델 제공업체(LLM) 목록 및 개요"
read_when:
  - 어떤 인공지능 모델을 사용할지 고민될 때
  - 지원되는 LLM 백엔드 제공업체를 한눈에 확인하고 싶을 때
title: "모델 제공업체"
---

# 모델 제공업체 (Model Providers)

OpenClaw는 수많은 LLM(대형 언어 모델) 제공업체를 지원합니다. 원하는 업체를 선택하고 인증을 마친 후, 기본 모델을 `제공업체/모델명` 형식으로 설정하여 사용할 수 있습니다.

*채팅 채널(WhatsApp/Telegram 등) 문서를 찾으시나요? [채널 가이드](/channels)를 확인하세요.*

## 주요 추천: 베니스 (Venice AI)
프라이버시를 중시하면서도 강력한 성능이 필요하다면 베니스(Venice AI)를 권장합니다.
- 기본 권장: `venice/llama-3.3-70b`
- 성능 우선: `venice/claude-opus-45`

상세 내용은 [베니스 가이드](/providers/venice)를 참고하세요.

## 빠른 시작
1.  **인증**: `openclaw onboard` 명령어를 통해 제공업체 인증을 진행합니다.
2.  **모델 설정**: `openclaw.json` 파일에서 기본 모델을 지정합니다.
    ```json5
    {
      "agents": { "defaults": { "model": { "primary": "anthropic/claude-opus-4-6" } } }
    }
    ```

## 지원 제공업체 목록
- [Anthropic (Claude)](/providers/anthropic)
- [OpenAI (GPT-4o)](/providers/openai)
- [Qwen (통의천문)](/providers/qwen)
- [OpenRouter (통합 허브)](/providers/openrouter)
- [LiteLLM (통합 게이트웨이)](/providers/litellm)
- [Moonshot AI (Kimi)](/providers/moonshot)
- [Mistral](/providers/mistral)
- [MiniMax](/providers/minimax)
- [Hugging Face (인퍼런스)](/providers/huggingface)
- [Ollama (로컬 모델)](/providers/ollama)
- [Amazon Bedrock](/providers/bedrock)
- 그 외 다수 (NVIDIA, Together AI, vLLM 등)

## 음성 인식(STT) 제공업체
- [Deepgram](/providers/deepgram)

## 커뮤니티 도구
- [Claude Max API Proxy](/providers/claude-max-api-proxy) : 클로드 맥스/프로 구독을 OpenAI API처럼 활용

---
더 상세한 카탈로그와 고급 설정은 [개념: 모델 제공업체](/concepts/model-providers) 문서를 확인하세요.
---
