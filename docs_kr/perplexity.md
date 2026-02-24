---
summary: "web_search 도구를 위한 Perplexity Sonar 설정 가이드"
read_when:
  - 웹 검색 엔진으로 Perplexity Sonar를 사용하고 싶을 때
  - PERPLEXITY_API_KEY 또는 OpenRouter 설정이 필요할 때
title: "Perplexity Sonar"
---

# Perplexity Sonar

OpenClaw는 웹 검색(`web_search`) 도구에 Perplexity Sonar를 사용할 수 있습니다. Perplexity의 공식 API를 직접 사용하거나 OpenRouter를 통해 연결할 수 있습니다.

## 연결 옵션

### 1. Perplexity 직접 연결
- 베이스 URL: [https://api.perplexity.ai](https://api.perplexity.ai)
- 환경 변수: `PERPLEXITY_API_KEY`

### 2. OpenRouter를 통한 연결
- 베이스 URL: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
- 환경 변수: `OPENROUTER_API_KEY`
- 선불 크레딧 또는 암호화폐 결제를 지원합니다.

## 설정 예시 (openclaw.json)

```json5
{
  "tools": {
    "web": {
      "search": {
        "provider": "perplexity",
        "perplexity": {
          "apiKey": "pplx-...", // 직접 발급받은 키
          "baseUrl": "https://api.perplexity.ai",
          "model": "perplexity/sonar-pro" // 사용할 모델 지정
        }
      }
    }
  }
}
```

## 사용 가능 모델

- `perplexity/sonar`: 웹 검색과 함께 제공되는 빠른 질의응답 모델
- `perplexity/sonar-pro` (기본값): 다단계 추론 및 웹 검색 지원
- `perplexity/sonar-reasoning-pro`: 심층 리서치 및 추론 모델

모든 웹 관련 도구 설정에 대한 내용은 [웹 도구 가이드](/tools/web)에서 확인할 수 있습니다.
---
