---
summary: "Kilo 게이트웨이의 통합 API를 사용하여 OpenClaw에서 다양한 모델에 접속하는 방법"
read_when:
  - 하나의 API 키로 여러 대형 언어 모델(LLM)에 접근하고 싶을 때
  - Kilo 게이트웨이를 통해 모델을 실행하고 싶을 때
title: "Kilo 게이트웨이 (Kilo Gateway)"
---

# Kilo 게이트웨이 (Kilo Gateway)

Kilo 게이트웨이는 단일 엔드포인트와 하나의 API 키로 수많은 모델에 요청을 라우팅할 수 있는 **통합 API**를 제공합니다. OpenAI 규격을 완벽히 지원하므로, 베이스 URL만 변경하여 간편하게 연동할 수 있습니다.

## API 키 발급 방법
1.  [app.kilo.ai](https://app.kilo.ai)에 접속하여 로그인합니다.
2.  API Keys 메뉴로 이동하여 새 키를 생성합니다.

## 빠른 설정 (CLI)
```bash
openclaw onboard --kilocode-api-key <본인의_API_키>
```
또는 환경 변수(`KILOCODE_API_KEY`)를 직접 설정할 수도 있습니다.

## 설정 예시 (`openclaw.json`)
```json5
{
  "env": { "KILOCODE_API_KEY": "sk-..." },
  "agents": {
    "defaults": {
      "model": { "primary": "kilocode/anthropic/claude-opus-4.6" } // 기본 권장 모델
    }
  }
}
```

## 주요 모델 식별자 (Refs)
Kilo 게이트웨이를 통해 아래와 같은 모델들을 바로 사용할 수 있습니다:
- `kilocode/anthropic/claude-opus-4.6` (기본 모델)
- `kilocode/openai/gpt-5.2`
- `kilocode/google/gemini-3-pro-preview`
- `kilocode/minimax/minimax-m2.5:free`
- `kilocode/moonshotai/kimi-k2.5`
- `kilocode/x-ai/grok-code-fast-1`

## 참고
- **형식**: 모델 식별자는 `kilocode/제공업체/모델명` 형식을 따릅니다.
- **엔드포인트**: 기본 주소는 `https://api.kilo.ai/api/gateway/`입니다.

---
더 많은 모델 옵션은 [모델 제공업체 개념 설명](/concepts/model-providers)을 참고하세요.
---
