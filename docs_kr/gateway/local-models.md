---
summary: "로컬 LLM(LM Studio, vLLM, LiteLLM 등)을 활용한 OpenClaw 실행 가이드"
read_when:
  - 개인 GPU 서버를 활용하여 모델을 서비스하고 싶을 때
  - LM Studio나 OpenAI 호환 프록시를 연결할 때
title: "로컬 모델 활용"
---

# 로컬 모델 (Local Models)

OpenClaw는 로컬 환경에서 실행되는 오픈소스 모델(LLM)을 지원합니다. 하지만 에이전트 기능을 제대로 활용하려면 큰 컨텍스트 윈도우와 강력한 성능이 필요합니다. 원활한 사용을 위해 고성능 GPU(Mac Studio 2대급 또는 동등한 GPU 리그)를 권장하며, 최소 24GB 이상의 VRAM을 갖춘 GPU를 사용해야 지연 시간을 줄일 수 있습니다.

## 권장 조합: LM Studio + MiniMax M2.1

현재 가장 추천되는 로컬 구성은 LM Studio에서 MiniMax M2.1 모델을 로드하여 사용하는 방식입니다.

### 설정 방법 (`openclaw.json`)

```json5
{
  "agents": {
    "defaults": {
      "model": { "primary": "lmstudio/minimax-m2.1-gs32" }
    }
  },
  "models": {
    "mode": "merge", // 기존 클라우드 모델과 병합하여 사용
    "providers": {
      "lmstudio": {
        "baseUrl": "http://127.0.0.1:1234/v1",
        "apiKey": "lmstudio",
        "api": "openai-responses", // 추론 과정과 답변 텍스트를 분리하는 API 형식
        "models": [
          {
            "id": "minimax-m2.1-gs32",
            "name": "MiniMax M2.1 GS32",
            "contextWindow": 196608,
            "maxTokens": 8192
          }
        ]
      }
    }
  }
}
```

### 준비 사항
- LM Studio 설치: [https://lmstudio.ai](https://lmstudio.ai)
- LM Studio에서 지원하는 가장 큰 MiniMax M2.1 빌드를 다운로드하고 로컬 서버 기능을 활성화하세요.
- `http://127.0.0.1:1234/v1/models` 엔드포인트가 정상적으로 정보를 반환하는지 확인하십시오.

## 응용 구성

### 1. 클라우드 모델을 메인으로, 로컬을 백업으로
안정적인 클라우드 API(Anthropic 등)를 주 모델로 쓰고, API 장애 발생 시 로컬 모델이 대처하도록 설정할 수 있습니다.

```json5
"model": {
  "primary": "anthropic/claude-3-5-sonnet",
  "fallbacks": ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-3-opus"]
}
```

### 2. 기타 로컬 프록시 (vLLM, LiteLLM 등)
OpenAI 호환 `/v1` 엔드포인트를 제공하는 모든 도구를 연결할 수 있습니다. `baseUrl`과 `modelId`만 해당 도구에 맞춰 수정하십시오.

## 주의 사항 및 팁

- **지연 시간**: 모델이 메모리에서 내려가면 다시 로드하는 데 시간이 걸립니다. 항상 모델을 로드된 상태로 유지하는 것이 좋습니다.
- **컨텍스트 제한**: 로컬 서버의 메모리 용량에 맞춰 `contextWindow` 값을 조절하십시오. 너무 큰 값은 에러를 유발할 수 있습니다.
- **보안**: 로컬 모델은 서비스 제공자의 필터링이 없으므로 프롬프트 인젝션 등에 더 취약할 수 있습니다. 에이전트의 권한 범위를 적절히 제한하십시오.

상세한 트러블슈팅과 기술 사양은 [영문 원본 문서](/gateway/local-models)를 참고하십시오.
---
