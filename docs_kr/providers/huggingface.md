---
summary: "허깅페이스(Hugging Face) 인퍼런스 서버 설정 가이드 (인증 및 모델 선택)"
read_when:
  - Hugging Face의 다양한 모델을 OpenClaw와 연동하고 싶을 때
  - HF 토큰(Token) 설정 및 인퍼런스 제공업체(Inference Providers) 기능을 확인해야 할 때
title: "허깅페이스 인퍼런스 (Hugging Face Inference)"
---

# 허깅페이스 인퍼런스 (Hugging Face Inference)

허깅페이스(Hugging Face)는 단일 라이우터 API를 통해 DeepSeek, Llama, Qwen 등 수많은 모델에 대한 OpenAI 호환 엔드포인트를 제공합니다. 하나의 토큰으로 수백 개의 최신 모델을 즉시 사용할 수 있는 강력한 플랫폼입니다.

## 빠른 설정 방법

1.  **토큰 생성**: [HF 설정 페이지](https://huggingface.co/settings/tokens)에서 **Make calls to Inference Providers** 권한이 포함된 Fine-grained 토큰을 생성합니다.
2.  **온보딩 실행**:
    ```bash
    openclaw onboard --auth-choice huggingface-api-key
    ```
    *안내에 따라 복사한 API 키를 입력하면 사용 가능한 모델 목록을 자동으로 가져옵니다.*
3.  **기본 모델 설정**: 설정 파일(`openclaw.json`)에서 원하는 모델을 지정할 수 있습니다.
    ```json5
    {
      "agents": {
        "defaults": {
          "model": { "primary": "huggingface/deepseek-ai/DeepSeek-R1" }
        }
      }
    }
    ```

## 주요 기능 및 정책
- **모델 식별자(Refs)**: `huggingface/조직명/모델명` 형식을 사용합니다.
- **라우팅 옵션**: 모델명 뒤에 접미사를 붙여 응답 방식을 제어할 수 있습니다.
  - `:fastest` — 가장 빠른 응답 속도를 가진 업체를 선택
  - `:cheapest` — 가장 저렴한 비용의 업체를 선택 (무료 티어 우선)
  - `:together`, `:sambanova` 등 — 특정 백엔드 업체를 강제로 지정
- **자동 갱신**: OpenClaw는 실행 시점에 HF API를 호출하여 최신 모델 카탈로그를 자동으로 업데이트합니다.

## 참고 사항
- **환경 변수**: `HUGGINGFACE_HUB_TOKEN` 또는 `HF_TOKEN` 변수를 지원합니다.
- **과금**: 하나의 HF 토큰으로 통합 결제가 가능하며, 제공업체별 요금 정책을 따릅니다.

---
더 많은 모델 목록은 [허깅페이스 모델 허브](https://huggingface.co/models)를 참고하세요.
---
