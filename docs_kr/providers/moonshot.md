---
summary: "Moonshot K2 및 Kimi Coding 설정 방법 (별도 제공업체 및 키 관리)"
read_when:
  - Moonshot K2(Moonshot Open Platform) 또는 Kimi Coding을 설정하고 싶을 때
  - 각각의 엔드포인트, API 키 및 모델 식별값을 확인해야 할 때
title: "문샷 AI (Moonshot AI / Kimi)"
---

# 문샷 AI (Moonshot AI / Kimi)

문샷(Moonshot)은 OpenAI와 호환되는 엔드포인트를 가진 Kimi API를 제공합니다. `moonshot/kimi-k2.5`와 같은 모델을 사용하거나, 코딩에 최적화된 `kimi-coding/k2p5` 모델을 사용할 수 있습니다.

## 중요 사항
Moonshot(일반)과 Kimi Coding은 **서로 다른 제공업체**로 취급됩니다. API 키와 엔드포인트 주소가 서로 다르며 호환되지 않으므로 주의하십시오.

## 모델 식별자 (Refs)
- **Moonshot**: `moonshot/kimi-k2.5`, `moonshot/kimi-k2-thinking` 등
- **Kimi Coding**: `kimi-coding/k2p5` 등

## 빠른 설정 (CLI)
```bash
# 일반 Moonshot API
openclaw onboard --auth-choice moonshot-api-key

# Kimi Coding 전용
openclaw onboard --auth-choice kimi-code-api-key
```

## 설정 예시 (Moonshot API)
```json5
{
  "env": { "MOONSHOT_API_KEY": "sk-..." },
  "agents": {
    "defaults": {
      "model": { "primary": "moonshot/kimi-k2.5" }
    }
  },
  "models": {
    "providers": {
      "moonshot": {
        "baseUrl": "https://api.moonshot.ai/v1",
        "apiKey": "${MOONSHOT_API_KEY}"
      }
    }
  }
}
```

## 참고
- **엔드포인트**: 글로벌 사용자는 `https://api.moonshot.ai/v1`, 중국 내 사용자는 `https://api.moonshot.cn/v1`을 사용합니다.
- **Thinking 모델**: `kimi-k2-thinking` 모델은 추론(Reasoning) 기능을 지원합니다.

---
다른 모델 제공업체에 대한 정보는 [모델 제공업체 개요](/providers/index)를 확인하세요.
---
