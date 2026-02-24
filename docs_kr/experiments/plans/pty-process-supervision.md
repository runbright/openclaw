---
summary: "명시적 소유권, 통합 생명주기, 결정적 정리를 갖춘 신뢰성 있는 대화형 프로세스 감독(PTY + 비-PTY)을 위한 프로덕션 계획"
read_when:
  - exec/프로세스 생명주기 소유권 및 정리 작업 시
  - PTY 및 비-PTY 감독 동작 디버깅 시
owner: "openclaw"
status: "in-progress"
last_updated: "2026-02-15"
title: "PTY 및 프로세스 감독 계획"
---

# PTY 및 프로세스 감독 계획

## 1. 문제 및 목표

다음 전반에 걸쳐 장기 실행 명령 실행을 위한 하나의 신뢰성 있는 생명주기가 필요합니다:

- `exec` 포그라운드 실행
- `exec` 백그라운드 실행
- `process` 후속 작업 (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
- CLI 에이전트 러너 서브프로세스

목표는 단순히 PTY를 지원하는 것이 아닙니다. 목표는 안전하지 않은 프로세스 매칭 휴리스틱 없이 예측 가능한 소유권, 취소, 타임아웃 및 정리입니다.

## 2. 범위 및 경계

- `src/process/supervisor` 내부 구현 유지.
- 이를 위한 새 패키지를 만들지 않음.
- 실용적인 범위에서 현재 동작 호환성 유지.
- 터미널 리플레이 또는 tmux 스타일 세션 지속성으로 범위를 확장하지 않음.

## 3. 이 브랜치에서 구현된 내용

### 이미 존재하는 감독자 기본선

- 감독자 모듈이 `src/process/supervisor/*` 아래에 위치함.
- Exec 런타임과 CLI 러너가 이미 감독자 spawn 및 wait를 통해 라우팅됨.
- 레지스트리 최종화가 멱등임.

### 이번 패스에서 완료된 항목

1. 명시적 PTY 명령 계약

- `SpawnInput`은 이제 `src/process/supervisor/types.ts`에서 구분된 유니온입니다.
- PTY 실행은 일반 `argv`를 재사용하는 대신 `ptyCommand`를 요구합니다.
- 감독자는 더 이상 `src/process/supervisor/supervisor.ts`에서 argv 결합으로 PTY 명령 문자열을 재구성하지 않습니다.
- Exec 런타임은 이제 `src/agents/bash-tools.exec-runtime.ts`에서 직접 `ptyCommand`를 전달합니다.

2. 프로세스 레이어 타입 분리

- 감독자 타입은 더 이상 에이전트에서 `SessionStdin`을 임포트하지 않습니다.
- 프로세스 로컬 stdin 계약은 `src/process/supervisor/types.ts` (`ManagedRunStdin`)에 있습니다.
- 어댑터는 이제 프로세스 수준 타입에만 의존합니다:
  - `src/process/supervisor/adapters/child.ts`
  - `src/process/supervisor/adapters/pty.ts`

3. 프로세스 도구 생명주기 소유권 개선

- `src/agents/bash-tools.process.ts`는 이제 먼저 감독자를 통해 취소를 요청합니다.
- `process kill/remove`는 이제 감독자 조회가 실패할 때 프로세스 트리 폴백 종료를 사용합니다.
- `remove`는 종료가 요청된 직후 실행 중인 세션 항목을 즉시 삭제하여 결정적 제거 동작을 유지합니다.

4. 단일 소스 워치독 기본값

- `src/agents/cli-watchdog-defaults.ts`에 공유 기본값 추가.
- `src/agents/cli-backends.ts`가 공유 기본값을 사용함.
- `src/agents/cli-runner/reliability.ts`가 동일한 공유 기본값을 사용함.

5. 불필요한 헬퍼 정리

- `src/agents/bash-tools.shared.ts`에서 사용되지 않는 `killSession` 헬퍼 경로 제거.

6. 직접 감독자 경로 테스트 추가

- 감독자 취소를 통한 kill 및 remove 라우팅을 다루는 `src/agents/bash-tools.process.supervisor.test.ts` 추가.

7. 신뢰성 갭 수정 완료

- `src/agents/bash-tools.process.ts`는 이제 감독자 조회가 실패할 때 실제 OS 수준 프로세스 종료로 폴백.
- `src/process/supervisor/adapters/child.ts`는 이제 기본 취소/타임아웃 종료 경로에 프로세스 트리 종료 시맨틱을 사용.
- `src/process/kill-tree.ts`에 공유 프로세스 트리 유틸리티 추가.

8. PTY 계약 엣지 케이스 커버리지 추가

- 그대로 전달되는 PTY 명령 및 빈 명령 거부를 위한 `src/process/supervisor/supervisor.pty-command.test.ts` 추가.
- child 어댑터 취소 시 프로세스 트리 종료 동작을 위한 `src/process/supervisor/adapters/child.test.ts` 추가.

## 4. 남은 갭 및 결정

### 신뢰성 상태

이번 패스에서 필요한 두 가지 신뢰성 갭이 해소되었습니다:

- `process kill/remove`에 이제 감독자 조회가 실패할 때 실제 OS 종료 폴백이 있음.
- child 취소/타임아웃이 이제 기본 종료 경로에 프로세스 트리 종료 시맨틱을 사용.
- 두 동작에 대한 회귀 테스트가 추가됨.

### 내구성 및 시작 조정

재시작 동작은 이제 인메모리 생명주기 전용으로 명시적으로 정의됩니다.

- `reconcileOrphans()`는 설계상 `src/process/supervisor/supervisor.ts`에서 no-op으로 유지됩니다.
- 프로세스 재시작 후 활성 실행이 복구되지 않습니다.
- 이 경계는 부분 지속성 위험을 피하기 위해 이번 구현 패스에서 의도적입니다.

### 유지보수 후속 작업

1. `src/agents/bash-tools.exec-runtime.ts`의 `runExecProcess`는 여전히 여러 책임을 처리하며 후속 작업에서 집중된 헬퍼로 분리할 수 있습니다.

## 5. 구현 계획

필요한 신뢰성 및 계약 항목에 대한 구현 패스가 완료되었습니다.

완료됨:

- `process kill/remove` 폴백 실제 종료
- child 어댑터 기본 종료 경로에 대한 프로세스 트리 취소
- 폴백 종료 및 child 어댑터 종료 경로에 대한 회귀 테스트
- 명시적 `ptyCommand` 하의 PTY 명령 엣지 케이스 테스트
- `reconcileOrphans()` no-op 설계로 명시적 인메모리 재시작 경계

선택적 후속 작업:

- `runExecProcess`를 동작 드리프트 없이 집중된 헬퍼로 분리

## 6. 파일 맵

### 프로세스 감독자

- `src/process/supervisor/types.ts` 구분된 spawn 입력 및 프로세스 로컬 stdin 계약으로 업데이트됨.
- `src/process/supervisor/supervisor.ts` 명시적 `ptyCommand`를 사용하도록 업데이트됨.
- `src/process/supervisor/adapters/child.ts` 및 `src/process/supervisor/adapters/pty.ts` 에이전트 타입에서 분리됨.
- `src/process/supervisor/registry.ts` 멱등 최종화 변경 없이 유지됨.

### Exec 및 프로세스 통합

- `src/agents/bash-tools.exec-runtime.ts` PTY 명령을 명시적으로 전달하고 폴백 경로를 유지하도록 업데이트됨.
- `src/agents/bash-tools.process.ts` 실제 프로세스 트리 폴백 종료를 포함하여 감독자를 통해 취소하도록 업데이트됨.
- `src/agents/bash-tools.shared.ts` 직접 종료 헬퍼 경로 제거됨.

### CLI 신뢰성

- `src/agents/cli-watchdog-defaults.ts` 공유 기본선으로 추가됨.
- `src/agents/cli-backends.ts` 및 `src/agents/cli-runner/reliability.ts`가 이제 동일한 기본값을 사용.

## 7. 이번 패스의 검증 실행

단위 테스트:

- `pnpm vitest src/process/supervisor/registry.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
- `pnpm vitest src/process/supervisor/adapters/child.test.ts`
- `pnpm vitest src/agents/cli-backends.test.ts`
- `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
- `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
- `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
- `pnpm vitest src/process/exec.test.ts`

E2E 대상:

- `pnpm vitest src/agents/cli-runner.test.ts`
- `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

타입체크 참고:

- 이 저장소에서 `pnpm build` (전체 lint/docs 게이트의 경우 `pnpm check`) 사용. `pnpm tsgo`를 언급하는 이전 노트는 더 이상 유효하지 않습니다.

## 8. 보존된 운영 보장

- Exec env 강화 동작은 변경되지 않음.
- 승인 및 허용 목록 흐름은 변경되지 않음.
- 출력 정제 및 출력 한도는 변경되지 않음.
- PTY 어댑터는 여전히 강제 종료 시 wait 정착 및 리스너 해제를 보장함.

## 9. 완료 정의

1. 감독자가 관리 실행의 생명주기 소유자임.
2. PTY spawn이 argv 재구성 없이 명시적 명령 계약을 사용함.
3. 프로세스 레이어가 감독자 stdin 계약에 대해 에이전트 레이어에 타입 의존성이 없음.
4. 워치독 기본값이 단일 소스임.
5. 대상 단위 및 e2e 테스트가 녹색을 유지함.
6. 재시작 내구성 경계가 명시적으로 문서화되거나 완전히 구현됨.

## 10. 요약

브랜치는 이제 일관되고 더 안전한 감독 형태를 갖추고 있습니다:

- 명시적 PTY 계약
- 더 깔끔한 프로세스 레이어링
- 프로세스 작업을 위한 감독자 주도 취소 경로
- 감독자 조회가 실패할 때의 실제 폴백 종료
- child 실행 기본 종료 경로를 위한 프로세스 트리 취소
- 통합된 워치독 기본값
- 명시적 인메모리 재시작 경계 (이번 패스에서 재시작 간 고아 조정 없음)
