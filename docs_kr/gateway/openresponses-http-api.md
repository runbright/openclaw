---
summary: "게이트웨이에서 OpenResponses 호환 /v1/responses HTTP 엔드포인트를 제공하는 방법 가이드"
read_when:
  - OpenResponses API 규격을 사용하는 클라이언트를 통합할 때
  - 아이템 기반 입력, 클라이언트 측 도구 호출, SSE 이벤트 등이 필요할 때
title: "OpenResponses API"
---

# OpenResponses API (HTTP)

OpenClaw 게이트웨이는 OpenResponses 규격과 호환되는 `POST /v1/responses` 엔드포인트를 제공할 수 있습니다. 이는 단순 텍스트를 넘어 이미지, 파일, 클라이언트 측 도구 호출 등을 포함한 복잡한 상호작용에 특화된 API입니다.

이 엔드포인트는 **기본적으로 비활성화**되어 있습니다. 사용하려면 설정에서 활성화해야 합니다.

- **URL**: `http://<게이트웨이-호스트>:<포트>/v1/responses`
- **방식**: `POST`

## 인증 및 에이전트 선택

인증과 에이전트 선택 방식은 [OpenAI 호환 API](/gateway/openai-http-api)와 동일합니다.
- **인증**: `Authorization: Bearer <토큰>`
- **에이전트**: 요청 바디의 `"model": "openclaw:<에이전트ID>"` 또는 헤더의 `x-openclaw-agent-id: <에이전트ID>` 사용.

## 주요 기능 및 지원 항목

### 1. 아이템 기반 입력 (`input`)
문자열뿐만 아니라 메시지 객체 배열을 통해 대화 맥락을 전달할 수 있습니다.
- **역할**: `system`, `user`, `assistant` 등 지원.
- **기능**: 이전 대화 기록을 포함하여 응답의 일관성을 유지합니다.

### 2. 멀티미디어 지원
- **이미지 (`input_image`)**: base64 데이터나 외부 URL을 통해 이미지를 전달할 수 있습니다. (JPEG, PNG, GIF, WebP 지원)
- **파일 (`input_file`)**: 텍스트, 마크다운, PDF 등을 지원합니다. PDF의 경우 텍스트를 추출하며, 텍스트가 부족할 경우 이미지로 래스터화하여 분석합니다.

### 3. 클라이언트 측 도구 (Function Tools)
클라이언트가 실행 가능한 도구 목록을 전달하고, 모델이 필요할 때 해당 도구의 실행을 요청(`function_call`)할 수 있습니다. 실행 결과는 다시 게이트웨이에 전달하여 대화를 이어나갑니다.

### 4. 실시간 스트리밍 (SSE)
`stream: true`를 설정하면 답변 생성 과정을 실시간 이벤트로 받아볼 수 있습니다.
- `response.output_text.delta`: 텍스트 답변의 실시간 조각
- `response.completed`: 전체 응답 완료

## 기능 설정 예시 (`openclaw.json`)

```json5
{
  "gateway": {
    "http": {
      "endpoints": {
        "responses": {
          "enabled": true,
          "images": { "maxBytes": 10485760 }, // 이미지 최대 10MB
          "files": { "maxBytes": 5242880 } // 파일 최대 5MB
        }
      }
    }
  }
}
```

## 사용 예시 (`curl`)

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "이 파일의 내용을 요약해줘.",
    "stream": true
  }'
```

상세한 파일 형식 제한이나 보안 정책은 [영문 원본 문서](/gateway/openresponses-http-api)를 참고하십시오.
---
