---
summary: "메뉴 바 상태 로직 및 사용자에게 표시되는 내용"
read_when:
  - macOS 메뉴 UI 또는 상태 로직 조정
title: "메뉴 바"
---

# 메뉴 바 상태 로직

## 표시 내용

- 현재 에이전트 작업 상태를 메뉴 바 아이콘과 메뉴의 첫 번째 상태 행에 표시합니다.
- 작업이 활성화된 동안 상태 확인은 숨겨지며, 모든 세션이 유휴 상태가 되면 다시 나타납니다.
- 메뉴의 "Nodes" 블록은 클라이언트/프레즌스 항목이 아닌 **기기**만 나열합니다 (`node.list`를 통한 페어링된 노드).
- 제공업체 사용량 스냅샷이 사용 가능할 때 Context 아래에 "Usage" 섹션이 나타납니다.

## 상태 모델

- 세션: 이벤트가 `runId` (실행별)와 페이로드의 `sessionKey`로 도착합니다. "main" 세션은 키 `main`입니다; 없으면 가장 최근에 업데이트된 세션으로 대체합니다.
- 우선순위: main이 항상 우선합니다. main이 활성이면 즉시 해당 상태가 표시됩니다. main이 유휴이면 가장 최근 활성화된 비main 세션이 표시됩니다. 활동 중에는 전환하지 않습니다; 현재 세션이 유휴 상태가 되거나 main이 활성화될 때만 전환합니다.
- 활동 종류:
  - `job`: 상위 수준 명령 실행 (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result`와 `toolName` 및 `meta/args`.

## IconState 열거형 (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (디버그 오버라이드)

### ActivityKind → 글리프

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- 기본값 → 🛠️

### 시각적 매핑

- `idle`: 일반 크리터.
- `workingMain`: 글리프가 있는 배지, 전체 틴트, 다리 "작업 중" 애니메이션.
- `workingOther`: 글리프가 있는 배지, 음소거된 틴트, 움직임 없음.
- `overridden`: 활동과 관계없이 선택한 글리프/틴트를 사용.

## 상태 행 텍스트 (메뉴)

- 작업이 활성화된 동안: `<세션 역할> · <활동 레이블>`
  - 예시: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- 유휴 시: 상태 확인 요약으로 대체.

## 이벤트 수집

- 소스: 제어 채널 `agent` 이벤트 (`ControlChannel.handleAgentEvent`).
- 파싱된 필드:
  - `stream: "job"`과 `data.state`로 시작/중지.
  - `stream: "tool"`과 `data.phase`, `name`, 선택사항 `meta`/`args`.
- 레이블:
  - `exec`: `args.command`의 첫 줄.
  - `read`/`write`: 축약된 경로.
  - `edit`: 경로와 `meta`/diff 카운트에서 추론한 변경 종류.
  - 대체: 도구 이름.

## 디버그 오버라이드

- Settings ▸ Debug ▸ "Icon override" 선택기:
  - `System (auto)` (기본값)
  - `Working: main` (도구 종류별)
  - `Working: other` (도구 종류별)
  - `Idle`
- `@AppStorage("iconOverride")`를 통해 저장; `IconState.overridden`으로 매핑.

## 테스트 체크리스트

- main 세션 작업 트리거: 아이콘이 즉시 전환되고 상태 행에 main 레이블이 표시되는지 확인.
- main 유휴 시 비main 세션 작업 트리거: 아이콘/상태가 비main을 표시; 완료될 때까지 안정 유지.
- 다른 활성 상태에서 main 시작: 아이콘이 즉시 main으로 전환.
- 빠른 도구 연속 실행: 배지가 깜빡이지 않는지 확인 (도구 결과에 TTL 유예).
- 모든 세션이 유휴 상태가 되면 상태 확인 행이 다시 나타남.
