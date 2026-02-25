---
summary: "OpenClaw에서 미스트랄(Mistral) 모델 및 Voxtral 음성 인식을 사용하는 방법"
read_when:
  - OpenClaw에 Mistral 모델을 연동하고 싶을 때
  - Mistral API 키 설정 및 오디오 전송 설정을 확인해야 할 때
title: "미스트랄 (Mistral)"
---

# 미스트랄 (Mistral)

OpenClaw는 텍스트/이미지 모델 라우팅(`mistral/...`)과 Voxtral을 통한 오디오 전송 기능을 지원합니다. 또한 메모리 임베딩(`mistral-embed`) 용도로도 사용할 수 있습니다.

## 빠른 설정 (CLI)
```bash
openclaw onboard --auth-choice mistral-api-key
# 또는 비대화형 모드
openclaw onboard --mistral-api-key "API_키_값"
```

## 설정 예시 (LLM 제공업체)
```json5
{
  "env": { "MISTRAL_API_KEY": "sk-..." },
  "agents": { "defaults": { "model": { "primary": "mistral/mistral-large-latest" } } }
}
```

## 주요 기능 및 정보
- **모델 연동**: 기본 모델로 `mistral/mistral-large-latest`를 권장합니다.
- **오디오 전송**: Mistral의 Voxtral 기능을 통해 음성 메시지를 텍스트로 변환할 수 있습니다. (`voxtral-mini-latest` 모델 사용)
- **메모리 기능**: `mistral-embed` 모델을 사용하여 에이전트의 기억(Embeddings) 기능을 구현할 수 있습니다.
- **인증**: `MISTRAL_API_KEY` 환경 변수를 사용합니다.

---
다른 모델 제공업체에 대한 정보는 [모델 제공업체 개요](/providers/index)를 확인하세요.
---
