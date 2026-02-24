---
summary: "`openclaw nodes` (목록/상태/승인/호출, 카메라/캔버스/화면) CLI 레퍼런스"
read_when:
  - 페어링된 노드(카메라, 화면, 캔버스)를 관리할 때
  - 요청을 승인하거나 노드 명령을 호출해야 할 때
title: "nodes"
---

# `openclaw nodes`

페어링된 노드(장치)를 관리하고 노드 기능을 호출합니다.

관련 항목:

- 노드 개요: [노드](/nodes)
- 카메라: [카메라 노드](/nodes/camera)
- 이미지: [이미지 노드](/nodes/images)

공통 옵션:

- `--url`, `--token`, `--timeout`, `--json`

## 일반 명령어

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list`는 대기 중/페어링된 테이블을 출력합니다. 페어링된 행에는 가장 최근 연결 시간(Last Connect)이 포함됩니다.
`--connected`를 사용하여 현재 연결된 노드만 표시합니다. `--last-connected <duration>`을 사용하여
지정된 기간 내에 연결된 노드를 필터링합니다 (예: `24h`, `7d`).

## 호출 / 실행

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

호출 플래그:

- `--params <json>`: JSON 객체 문자열 (기본값 `{}`).
- `--invoke-timeout <ms>`: 노드 호출 타임아웃 (기본값 `15000`).
- `--idempotency-key <key>`: 선택적 멱등성 키.

### 실행 스타일 기본값

`nodes run`은 모델의 실행 동작(기본값 + 승인)을 미러링합니다:

- `tools.exec.*` (및 `agents.list[].tools.exec.*` 재정의)를 읽습니다.
- `system.run`을 호출하기 전에 실행 승인(`exec.approval.request`)을 사용합니다.
- `tools.exec.node`가 설정된 경우 `--node`를 생략할 수 있습니다.
- `system.run`을 알리는 노드(macOS 컴패니언 앱 또는 헤드리스 노드 호스트)가 필요합니다.

플래그:

- `--cwd <path>`: 작업 디렉토리.
- `--env <key=val>`: 환경 변수 재정의 (반복 가능). 참고: 노드 호스트는 `PATH` 재정의를 무시합니다 (`tools.exec.pathPrepend`는 노드 호스트에 적용되지 않음).
- `--command-timeout <ms>`: 명령 타임아웃.
- `--invoke-timeout <ms>`: 노드 호출 타임아웃 (기본값 `30000`).
- `--needs-screen-recording`: 화면 녹화 권한 필요.
- `--raw <command>`: 셸 문자열 실행 (`/bin/sh -lc` 또는 `cmd.exe /c`).
  Windows 노드 호스트의 허용 목록 모드에서 `cmd.exe /c` 셸 래퍼 실행은 승인이 필요합니다
  (허용 목록 항목만으로는 래퍼 형식이 자동 허용되지 않음).
- `--agent <id>`: 에이전트 범위의 승인/허용 목록 (기본값은 구성된 에이전트).
- `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: 재정의.
