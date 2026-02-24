---
summary: "온보딩 마법사 및 설정 스키마 엔드포인트에 대한 RPC 프로토콜 노트"
read_when: "온보딩 마법사 단계 또는 설정 스키마 엔드포인트를 변경할 때"
title: "온보딩 및 설정 프로토콜"
---

# 온보딩 + 설정 프로토콜

목적: CLI, macOS 앱, Web UI 전반에 걸친 공유 온보딩 + 설정 화면.

## 구성 요소

- 마법사 엔진 (공유 세션 + 프롬프트 + 온보딩 상태).
- CLI 온보딩은 UI 클라이언트와 동일한 마법사 흐름을 사용합니다.
- Gateway RPC는 마법사 + 설정 스키마 엔드포인트를 노출합니다.
- macOS 온보딩은 마법사 단계 모델을 사용합니다.
- Web UI는 JSON Schema + UI 힌트에서 설정 양식을 렌더링합니다.

## Gateway RPC

- `wizard.start` 매개변수: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` 매개변수: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` 매개변수: `{ sessionId }`
- `wizard.status` 매개변수: `{ sessionId }`
- `config.schema` 매개변수: `{}`

응답 (형태)

- 마법사: `{ sessionId, done, step?, status?, error? }`
- 설정 스키마: `{ schema, uiHints, version, generatedAt }`

## UI 힌트

- `uiHints`는 경로별로 키가 지정됩니다; 선택적 메타데이터 (label/help/group/order/advanced/sensitive/placeholder).
- 민감한 필드는 비밀번호 입력으로 렌더링됩니다; 마스킹 레이어가 없습니다.
- 지원되지 않는 스키마 노드는 원시 JSON 편집기로 폴백됩니다.

## 참고사항

- 이 문서는 온보딩/설정에 대한 프로토콜 리팩토링을 추적하는 유일한 장소입니다.
