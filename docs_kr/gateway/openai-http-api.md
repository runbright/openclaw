---
summary: "게이트웨이에서 OpenAI 호환 /v1/chat/completions HTTP 엔드포인트를 제공하는 방법 가이드"
read_when:
  - OpenAI Chat Completions 형식을 요구하는 외부 도구와 통합할 때
title: "OpenAI 호환 HTTP API"
---

# OpenAI 호환 HTTP API (Chat Completions)

OpenClaw 게이트웨이는 OpenAI API 규격과 호환되는 텍스트 생성 엔드포인트를 제공할 수 있습니다. 이를 통해 OpenAI API를 지원하는 다양한 외부 도구나 라이브러리를 OpenClaw 에이전트에 연결할 수 있습니다.

이 엔드포인트는 **기본적으로 비활성화**되어 있습니다. 사용하려면 설정에서 활성화해야 합니다.

- **URL**: `http://<게이트웨이-호스트>:<포트>/v1/chat/completions`
- **방식**: `POST`

## 인증 (Authentication)

게이트웨이에 설정된 인증 정보를 사용하여 Bearer 토큰 방식으로 인증합니다.

```
Authorization: Bearer <게이트웨이_토큰>
```

## 에이전트 선택 방법

OpenAI 요청의 `model` 필드에 OpenClaw 에이전트 ID를 적거나, 헤더를 사용합니다.

- **방법 1 (모델 필드)**: `"model": "openclaw:main"` 또는 `"model": "agent:main"`
- **방법 2 (헤더)**: `x-openclaw-agent-id: main`

## 기능 활성화 설정 (`openclaw.json`)

```json5
{
  "gateway": {
    "http": {
      "endpoints": {
        "chatCompletions": { "enabled": true }
      }
    }
  }
}
```

## 주요 특징

1. **상태 유지 (Sessions)**: 기본적으로는 호출마다 독립된 세션으로 취급되지만, 요청 바디에 OpenAI `user` 필드(문자열)를 포함하면 해당 값을 기반으로 세션이 유지되어 연속된 대화가 가능합니다.
2. **스트리밍 (Streaming)**: `stream: true`를 설정하면 OpenAI 규격과 동일한 Server-Sent Events (SSE) 방식으로 답변을 실시간 전달받을 수 있습니다.
3. **통합 실행**: 이 API를 통한 요청은 `openclaw agent` 호출과 동일한 경로로 처리되므로, 당신이 설정한 모델 라우팅, 도구 사용, 보안 규칙 등이 그대로 적용됩니다.

## 요청 예시 (`curl`)

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"안녕"}]
  }'
```

상세한 에러 코드 및 기술 사양은 [영문 원본 문서](/gateway/openai-http-api)를 참고하십시오.
---
