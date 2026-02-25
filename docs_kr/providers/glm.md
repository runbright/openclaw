---
summary: "GLM 모델 시리즈 개요 및 OpenClaw 연동 방법"
read_when:
  - OpenClaw에서 GLM 모델을 사용하고 싶을 때
  - GLM 모델 네이밍 규칙 및 설정 방법을 확인해야 할 때
title: "GLM 모델"
---

# GLM 모델 (GLM Models)

GLM은 Z.AI 플랫폼을 통해 제공되는 **모델 시리즈**입니다. OpenClaw에서는 `zai` 제공업체를 통해 접근하며, `zai/glm-5`와 같은 모델 식별자를 사용합니다.

## 빠른 설정 (CLI)
```bash
openclaw onboard --auth-choice zai-api-key
```

## 설정 예시 (`openclaw.json`)
```json5
{
  "env": { "ZAI_API_KEY": "sk-..." },
  "agents": { "defaults": { "model": { "primary": "zai/glm-5" } } }
}
```

## 주요 정보
- **모델 종류**: `glm-5`, `glm-4.7`, `glm-4.6` 등 다양한 버전을 사용할 수 있습니다.
- **제공업체**: 상세한 연동 규칙은 [Z.AI 제공업체 가이드](/providers/zai)를 참고하세요.
- **최신성**: 사용 가능한 모델 버전은 Z.AI 플랫폼의 정책에 따라 변경될 수 있습니다.

---
다른 모델 제공업체에 대한 정보는 [모델 제공업체 개요](/providers/index)를 확인하세요.
---
