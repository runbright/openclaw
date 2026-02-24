---
summary: "모델 제공사별 설정 예시 및 CLI를 활용한 온보딩 가이드"
read_when:
  - 특정 모델 제공사(OpenAI, Anthropic, Google 등)의 설정 방법을 알고 싶을 때
  - API 키 순환(Rotation)이나 커스텀 엔드포인트 설정이 필요할 때
title: "모델 제공사 가이드"
---

# 모델 제공사 (Model Providers)

이 문서는 에이전트가 사용하는 **LLM 모델 제공사**에 대한 설정 가이드입니다. (WhatsApp, Telegram 같은 채팅 채널 설정이 아닙니다.)

## 기본 규칙

- 모델 식별자는 `제공사/모델ID` 형식을 사용합니다. (예: `openai/gpt-4o`)
- **CLI 도구 활용**: `openclaw onboard` 명령어를 통해 안내에 따라 쉽게 설정할 수 있습니다.
- **API 키 순환**: 동일한 제공사에 대해 여러 개의 API 키를 등록하면, 속도 제한이 걸렸을 때 다음 키로 자동 전환합니다.

## 주요 기본 제공사 (Built-in)

### 1. OpenAI
- **ID**: `openai`
- **인증**: `OPENAI_API_KEY` 환경 변수 또는 설정 파일
- **CLI**: `openclaw onboard --auth-choice openai-api-key`

### 2. Anthropic (Claude)
- **ID**: `anthropic`
- **인증**: `ANTHROPIC_API_KEY` 또는 전용 토큰
- **CLI**: `openclaw onboard --auth-choice token` (Claude Code 토큰 사용 시)

### 3. Google Gemini
- **ID**: `google`
- **인증**: `GEMINI_API_KEY` 환경 변수
- **CLI**: `openclaw onboard --auth-choice gemini-api-key`

### 4. OpenRouter
- **ID**: `openrouter`
- **인증**: `OPENROUTER_API_KEY`
- 다양한 오픈 소스 모델과 유료 모델을 통합하여 사용할 때 유용합니다.

## 커스텀 제공사 및 로컬 모델 (Local/Proxy)

### 1. Ollama (로컬 LLM)
로컬에서 실행 중인 Ollama를 활용할 수 있습니다.
- **ID**: `ollama`
- **인증**: 로컬 서버이므로 별도 키 불필요
- **사용법**: `ollama pull llama3.1` 실행 후 `ollama/llama3.1` 모델 지정

### 2. 커스텀 OpenAI 호환 엔드포인트
LM Studio, vLLM, LiteLLM 등 OpenAI 호환 API를 제공하는 모든 서비스를 연결할 수 있습니다.

```json5
{
  "models": {
    "providers": {
      "my-local-proxy": {
        "baseUrl": "http://localhost:1234/v1",
        "apiKey": "필요시_입력",
        "api": "openai-completions",
        "models": [{ "id": "my-model", "name": "Local Model" }]
      }
    }
  }
}
```

## API 키 순환 (Rotation) 설정

여러 개의 키를 사용하려면 쉼표나 세미콜론으로 구분하여 환경 변수를 설정하세요:
`OPENAI_API_KEYS="sk-key1,sk-key2,sk-key3"`

에이전트는 첫 번째 키로 시도하다가 `429 (Rate Limit)` 오류가 발생하면 자동으로 다음 키를 사용하여 작업을 재개합니다.

---
상세한 오류 복구 로직은 [모델 오류 복구 필오버](/concepts/model-failover) 문서를 참고하세요.
---
