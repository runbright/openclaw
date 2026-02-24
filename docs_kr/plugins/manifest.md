---
summary: "플러그인 매니페스트 + JSON 스키마 요구사항 (엄격한 설정 유효성 검사)"
read_when:
  - OpenClaw 플러그인을 빌드하는 경우
  - 플러그인 설정 스키마를 배포하거나 플러그인 유효성 검사 오류를 디버그하는 경우
title: "플러그인 매니페스트"
---

# 플러그인 매니페스트 (openclaw.plugin.json)

모든 플러그인은 **플러그인 루트**에 `openclaw.plugin.json` 파일을 **반드시** 포함해야 합니다.
OpenClaw는 이 매니페스트를 사용하여 **플러그인 코드를 실행하지 않고** 설정을 유효성 검사합니다.
누락되거나 유효하지 않은 매니페스트는 플러그인 오류로 처리되며 설정 유효성 검사를 차단합니다.

전체 플러그인 시스템 가이드: [플러그인](/tools/plugin).

## 필수 필드

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

필수 키:

- `id` (string): 정식 플러그인 ID.
- `configSchema` (object): 플러그인 설정을 위한 JSON Schema (인라인).

선택적 키:

- `kind` (string): 플러그인 종류 (예: `"memory"`).
- `channels` (array): 이 플러그인이 등록하는 채널 ID (예: `["matrix"]`).
- `providers` (array): 이 플러그인이 등록하는 프로바이더 ID.
- `skills` (array): 로드할 스킬 디렉토리 (플러그인 루트 기준 상대 경로).
- `name` (string): 플러그인 표시 이름.
- `description` (string): 짧은 플러그인 요약.
- `uiHints` (object): UI 렌더링을 위한 설정 필드 레이블/플레이스홀더/민감 플래그.
- `version` (string): 플러그인 버전 (정보성).

## JSON Schema 요구사항

- **모든 플러그인은 JSON Schema를 반드시 포함해야 합니다**, 설정을 받지 않더라도.
- 빈 스키마도 허용됩니다 (예: `{ "type": "object", "additionalProperties": false }`).
- 스키마는 런타임이 아닌 설정 읽기/쓰기 시점에 유효성 검사됩니다.

## 유효성 검사 동작

- 알 수 없는 `channels.*` 키는 플러그인 매니페스트가 해당 채널 ID를 선언하지 않는 한 **오류**입니다.
- `plugins.entries.<id>`, `plugins.allow`, `plugins.deny`, `plugins.slots.*`는
  **검색 가능한** 플러그인 ID를 참조해야 합니다. 알 수 없는 ID는 **오류**입니다.
- 플러그인이 설치되었지만 매니페스트나 스키마가 손상되었거나 누락된 경우,
  유효성 검사가 실패하고 Doctor가 플러그인 오류를 보고합니다.
- 플러그인 설정이 있지만 플러그인이 **비활성화**된 경우, 설정은 유지되고
  Doctor + 로그에 **경고**가 표시됩니다.

## 참고사항

- 매니페스트는 로컬 파일시스템 로드를 포함한 **모든 플러그인에 필수**입니다.
- 런타임은 여전히 플러그인 모듈을 별도로 로드합니다; 매니페스트는 검색 + 유효성 검사를 위한 것입니다.
- 플러그인이 네이티브 모듈에 의존하는 경우, 빌드 단계와 패키지 매니저 허용 목록 요구사항을
  문서화하세요 (예: pnpm `allow-build-scripts` - `pnpm rebuild <package>`).
