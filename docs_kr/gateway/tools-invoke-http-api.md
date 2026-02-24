---
summary: "게이트웨이 HTTP 엔드포인트를 통해 단일 도구를 직접 실행하는 방법 가이드"
read_when:
  - 에이전트를 통하지 않고 특정 도구만 바로 호출하고 싶을 때
  - 자급자족형 자동화 도구에서 OpenClaw의 도구 정책을 그대로 활용하고 싶을 때
title: "도구 직접 호출 API"
---

# 도구 직접 호출 API (Tools Invoke)

OpenClaw 게이트웨이는 에이전트의 사고 과정을 거치지 않고 특정 도구를 즉시 실행할 수 있는 간단한 HTTP 엔드포인트를 제공합니다. 이 기능은 항상 활성화되어 있으며, 게이트웨이 인증 및 도구 권한 정책(Policy)이 동일하게 적용됩니다.

- **URL**: `http://<게이트웨이-호스트>:<포트>/tools/invoke`
- **방식**: `POST`
- **최대 데이터 크기**: 기본 2MB

## 인증 방식

게이트웨이에 설정된 인증 정보를 사용하여 Bearer 토큰 방식으로 인증합니다.

```
Authorization: Bearer <게이트웨이_토큰>
```

## 요청 형식 (Request Body)

```json
{
  "tool": "sessions_list", // 실행할 도구 이름 (필수)
  "action": "json",        // 도구의 액션 (선택)
  "args": {},              // 도구에 전달할 인자 (선택)
  "sessionKey": "main"     // 대상 세션 키 (선택)
}
```

- **`tool`**: 호출할 도구의 이름입니다.
- **`sessionKey`**: 특정 세션의 맥락에서 도구를 실행하고 싶을 때 지정합니다. 생략하면 기본(`main`) 세션이 사용됩니다.

## 보안 및 권한 정책

도구 실행 가능 여부는 게이트웨이 에이전트와 동일한 정책 체인을 따릅니다. 사용 중인 에이전트 설정이나 글로벌 설정에서 허용(`allow`)되지 않은 도구는 실행할 수 없습니다.

### 추가 보안 제한 (Hard Deny)
보안을 위해 HTTP 직접 호출 방식에서는 다음 도구들이 기본적으로 차단됩니다:
- `sessions_spawn`, `sessions_send` (세션 제어 관련)
- `gateway`, `whatsapp_login` (시스템 관리 관련)

이 차단 목록을 수정하려면 `gateway.tools` 설정을 변경하십시오.

## 응답 형식

- **성공 (`200 OK`)**: `{ "ok": true, "result": ... }`
- **실패 (`400/401/404/500`)**: `{ "ok": false, "error": { "type": "...", "message": "..." } }`
  - 권한이 없어 실행할 수 없는 도구인 경우 **404** 에러를 반환합니다.

## 사용 예시 (`curl`)

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

상세한 에러 코드 정의는 [영문 원본 문서](/gateway/tools-invoke-http-api)를 참고하십시오.
---
