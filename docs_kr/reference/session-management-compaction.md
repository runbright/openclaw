---
summary: "심층 분석: 세션 저장소 + 트랜스크립트, 생명주기, (자동)컴팩션 내부"
read_when:
  - You need to debug session ids, transcript JSONL, or sessions.json fields
  - You are changing auto-compaction behavior or adding "pre-compaction" housekeeping
  - You want to implement memory flushes or silent system turns
title: "세션 관리 심층 분석"
---

# 세션 관리 및 컴팩션 (심층 분석)

이 문서는 OpenClaw이 세션을 엔드투엔드로 관리하는 방법을 설명합니다:

- **세션 라우팅** (인바운드 메시지가 `sessionKey`에 매핑되는 방법)
- **세션 저장소** (`sessions.json`) 및 추적하는 내용
- **트랜스크립트 영속성** (`*.jsonl`) 및 구조
- **트랜스크립트 위생** (실행 전 프로바이더별 수정)
- **컨텍스트 제한** (컨텍스트 윈도우 vs 추적된 토큰)
- **컴팩션** (수동 + 자동 컴팩션) 및 사전 컴팩션 작업을 연결하는 위치
- **사일런트 하우스키핑** (예: 사용자에게 보이는 출력을 생성하지 않아야 하는 메모리 쓰기)

상위 수준 개요를 먼저 보려면 다음을 시작하세요:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## 진실의 원천: 게이트웨이

OpenClaw은 세션 상태를 소유하는 단일 **게이트웨이 프로세스**를 중심으로 설계되었습니다.

- UI (macOS 앱, 웹 Control UI, TUI)는 세션 목록과 토큰 수를 위해 게이트웨이에 쿼리해야 합니다.
- 원격 모드에서 세션 파일은 원격 호스트에 있습니다; "로컬 Mac 파일 확인"은 게이트웨이가 사용하는 것을 반영하지 않습니다.

---

## 두 가지 영속성 계층

OpenClaw은 두 계층으로 세션을 영속화합니다:

1. **세션 저장소 (`sessions.json`)**
   - 키/값 맵: `sessionKey -> SessionEntry`
   - 작고, 변경 가능하며, 편집 (또는 항목 삭제)이 안전합니다
   - 세션 메타데이터를 추적합니다 (현재 세션 id, 마지막 활동, 토글, 토큰 카운터 등)

2. **트랜스크립트 (`<sessionId>.jsonl`)**
   - 트리 구조의 추가 전용 트랜스크립트 (항목에 `id` + `parentId` 포함)
   - 실제 대화 + 도구 호출 + 컴팩션 요약을 저장합니다
   - 향후 턴을 위한 모델 컨텍스트 재구성에 사용됩니다

---

## 디스크 위치

에이전트별, 게이트웨이 호스트:

- 저장소: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- 트랜스크립트: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram 토픽 세션: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw은 `src/config/sessions.ts`를 통해 이를 해석합니다.

---

## 저장소 유지보수 및 디스크 제어

세션 영속성에는 `sessions.json`과 트랜스크립트 아티팩트를 위한 자동 유지보수 제어 (`session.maintenance`)가 있습니다:

- `mode`: `warn` (기본값) 또는 `enforce`
- `pruneAfter`: 오래된 항목 연령 임계값 (기본값 `30d`)
- `maxEntries`: `sessions.json`의 항목 수 제한 (기본값 `500`)
- `rotateBytes`: 과대 시 `sessions.json` 로테이션 (기본값 `10mb`)
- `resetArchiveRetention`: `*.reset.<timestamp>` 트랜스크립트 아카이브 보존 기간 (기본값: `pruneAfter`와 동일; `false`는 정리 비활성화)
- `maxDiskBytes`: 선택적 세션 디렉토리 예산
- `highWaterBytes`: 정리 후 선택적 목표 (기본값 `maxDiskBytes`의 `80%`)

디스크 예산 정리 실행 순서 (`mode: "enforce"`):

1. 가장 오래된 아카이브 또는 고아 트랜스크립트 아티팩트를 먼저 제거.
2. 여전히 목표 이상이면 가장 오래된 세션 항목과 트랜스크립트 파일을 제거.
3. 사용량이 `highWaterBytes` 이하가 될 때까지 계속.

`mode: "warn"`에서는 OpenClaw이 잠재적 제거를 보고하지만 저장소/파일을 변경하지 않습니다.

필요시 유지보수 실행:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

---

## Cron 세션 및 실행 로그

격리된 cron 실행도 세션 항목/트랜스크립트를 생성하며, 전용 보존 제어가 있습니다:

- `cron.sessionRetention` (기본값 `24h`)은 세션 저장소에서 오래된 격리 cron 실행 세션을 정리합니다 (`false`는 비활성화).
- `cron.runLog.maxBytes` + `cron.runLog.keepLines`는 `~/.openclaw/cron/runs/<jobId>.jsonl` 파일을 정리합니다 (기본값: `2_000_000` 바이트 및 `2000` 줄).

---

## 세션 키 (`sessionKey`)

`sessionKey`는 _어떤 대화 버킷_에 있는지를 식별합니다 (라우팅 + 격리).

일반적인 패턴:

- 메인/다이렉트 채팅 (에이전트별): `agent:<agentId>:<mainKey>` (기본값 `main`)
- 그룹: `agent:<agentId>:<channel>:group:<id>`
- 방/채널 (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` 또는 `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (오버라이드하지 않는 한)

표준 규칙은 [/concepts/session](/concepts/session)에 문서화되어 있습니다.

---

## 세션 ID (`sessionId`)

각 `sessionKey`는 현재 `sessionId` (대화를 계속하는 트랜스크립트 파일)를 가리킵니다.

경험 법칙:

- **리셋** (`/new`, `/reset`)은 해당 `sessionKey`에 대해 새 `sessionId`를 생성합니다.
- **일일 리셋** (게이트웨이 호스트의 로컬 시간 기준 기본 오전 4:00)은 리셋 경계 이후 다음 메시지에서 새 `sessionId`를 생성합니다.
- **유휴 만료** (`session.reset.idleMinutes` 또는 레거시 `session.idleMinutes`)는 유휴 윈도우 이후 메시지가 도착하면 새 `sessionId`를 생성합니다. 일일 + 유휴가 모두 설정된 경우 먼저 만료되는 쪽이 적용됩니다.

구현 세부사항: `src/auto-reply/reply/session.ts`의 `initSessionState()`에서 결정됩니다.

---

## 세션 저장소 스키마 (`sessions.json`)

저장소의 값 타입은 `src/config/sessions.ts`의 `SessionEntry`입니다.

주요 필드 (전체 목록 아님):

- `sessionId`: 현재 트랜스크립트 id (`sessionFile`이 설정되지 않은 한 파일명이 여기서 파생됨)
- `updatedAt`: 마지막 활동 타임스탬프
- `sessionFile`: 선택적 명시 트랜스크립트 경로 오버라이드
- `chatType`: `direct | group | room` (UI 및 전송 정책에 도움)
- `provider`, `subject`, `room`, `space`, `displayName`: 그룹/채널 라벨링을 위한 메타데이터
- 토글:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (세션별 오버라이드)
- 모델 선택:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- 토큰 카운터 (최선 노력 / 프로바이더 의존):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: 이 세션 키에 대해 자동 컴팩션이 완료된 횟수
- `memoryFlushAt`: 마지막 사전 컴팩션 메모리 플러시 타임스탬프
- `memoryFlushCompactionCount`: 마지막 플러시가 실행된 시점의 컴팩션 카운트

저장소는 편집이 안전하지만, 게이트웨이가 권위자입니다: 세션이 실행되면서 항목을 다시 쓰거나 재수화할 수 있습니다.

---

## 트랜스크립트 구조 (`*.jsonl`)

트랜스크립트는 `@mariozechner/pi-coding-agent`의 `SessionManager`로 관리됩니다.

파일은 JSONL입니다:

- 첫 줄: 세션 헤더 (`type: "session"`, `id`, `cwd`, `timestamp`, 선택적 `parentSession` 포함)
- 이후: `id` + `parentId` (트리)가 있는 세션 항목

주요 항목 타입:

- `message`: user/assistant/toolResult 메시지
- `custom_message`: 확장에서 주입한 메시지로 모델 컨텍스트에 _포함됨_ (UI에서 숨길 수 있음)
- `custom`: 모델 컨텍스트에 _포함되지 않는_ 확장 상태
- `compaction`: `firstKeptEntryId`와 `tokensBefore`를 포함한 영속 컴팩션 요약
- `branch_summary`: 트리 브랜치 탐색 시 영속 요약

OpenClaw은 의도적으로 트랜스크립트를 "수정"하지 **않습니다**; 게이트웨이는 `SessionManager`를 사용하여 읽기/쓰기합니다.

---

## 컨텍스트 윈도우 vs 추적된 토큰

두 가지 다른 개념이 중요합니다:

1. **모델 컨텍스트 윈도우**: 모델별 하드 캡 (모델에 보이는 토큰)
2. **세션 저장소 카운터**: `sessions.json`에 기록된 롤링 통계 (/status 및 대시보드에 사용)

제한을 튜닝할 때:

- 컨텍스트 윈도우는 모델 카탈로그에서 가져옵니다 (설정으로 오버라이드 가능).
- 저장소의 `contextTokens`는 런타임 추정/보고 값입니다; 엄격한 보장으로 취급하지 마세요.

자세한 내용은 [/token-use](/reference/token-use)를 참조하세요.

---

## 컴팩션: 무엇인가

컴팩션은 오래된 대화를 트랜스크립트의 영속 `compaction` 항목으로 요약하고 최근 메시지는 그대로 유지합니다.

컴팩션 후 향후 턴에서 보이는 것:

- 컴팩션 요약
- `firstKeptEntryId` 이후의 메시지

컴팩션은 **영속적**입니다 (세션 프루닝과 다름). [/concepts/session-pruning](/concepts/session-pruning)을 참조하세요.

---

## 자동 컴팩션이 발생하는 시점 (Pi 런타임)

임베디드 Pi 에이전트에서 자동 컴팩션은 두 가지 경우에 트리거됩니다:

1. **오버플로우 복구**: 모델이 컨텍스트 오버플로우 에러를 반환 → 컴팩트 → 재시도.
2. **임계값 유지보수**: 성공적인 턴 이후, 다음 조건일 때:

`contextTokens > contextWindow - reserveTokens`

여기서:

- `contextWindow`는 모델의 컨텍스트 윈도우
- `reserveTokens`는 프롬프트 + 다음 모델 출력을 위해 예약된 여유 공간

이는 Pi 런타임 의미론입니다 (OpenClaw은 이벤트를 소비하지만 Pi가 컴팩션 시점을 결정합니다).

---

## 컴팩션 설정 (`reserveTokens`, `keepRecentTokens`)

Pi의 컴팩션 설정은 Pi 설정에 있습니다:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw은 임베디드 실행에 대한 안전 하한도 적용합니다:

- `compaction.reserveTokens < reserveTokensFloor`이면 OpenClaw이 올립니다.
- 기본 하한은 `20000` 토큰.
- `agents.defaults.compaction.reserveTokensFloor: 0`으로 하한 비활성화.
- 이미 높으면 OpenClaw이 그대로 둡니다.

이유: 컴팩션이 불가피해지기 전에 멀티턴 "하우스키핑" (예: 메모리 쓰기)을 위한 충분한 여유 공간 확보.

구현: `src/agents/pi-settings.ts`의 `ensurePiCompactionReserveTokens()`
(`src/agents/pi-embedded-runner.ts`에서 호출됨).

---

## 사용자에게 보이는 표면

다음을 통해 컴팩션과 세션 상태를 관찰할 수 있습니다:

- `/status` (모든 채팅 세션에서)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- 상세 모드: `Auto-compaction complete` + 컴팩션 카운트

---

## 사일런트 하우스키핑 (`NO_REPLY`)

OpenClaw은 사용자에게 중간 출력을 보여주지 않아야 하는 백그라운드 작업을 위한 "사일런트" 턴을 지원합니다.

규약:

- 어시스턴트가 출력을 `NO_REPLY`로 시작하여 "사용자에게 응답을 전달하지 마세요"를 나타냅니다.
- OpenClaw은 전달 계층에서 이를 제거/억제합니다.

`2026.1.10` 기준, OpenClaw은 부분 청크가 `NO_REPLY`로 시작할 때 **초안/타이핑 스트리밍**도 억제하여 사일런트 작업이 턴 중간에 부분 출력을 누출하지 않습니다.

---

## 사전 컴팩션 "메모리 플러시" (구현됨)

목표: 자동 컴팩션이 발생하기 전에 디스크에 지속적인 상태를 쓰는 사일런트 에이전틱 턴을 실행
(예: 에이전트 워크스페이스의 `memory/YYYY-MM-DD.md`)하여 컴팩션이 중요한 컨텍스트를 지울 수 없도록 합니다.

OpenClaw은 **사전 임계값 플러시** 접근 방식을 사용합니다:

1. 세션 컨텍스트 사용량을 모니터링.
2. "소프트 임계값" (Pi의 컴팩션 임계값 이하)을 넘으면 에이전트에게 사일런트
   "지금 메모리 쓰기" 지시를 실행.
3. 사용자에게 아무것도 보이지 않도록 `NO_REPLY` 사용.

설정 (`agents.defaults.compaction.memoryFlush`):

- `enabled` (기본값: `true`)
- `softThresholdTokens` (기본값: `4000`)
- `prompt` (플러시 턴의 사용자 메시지)
- `systemPrompt` (플러시 턴에 추가되는 추가 시스템 프롬프트)

참고사항:

- 기본 prompt/system prompt에는 전달 억제를 위한 `NO_REPLY` 힌트가 포함됩니다.
- 플러시는 컴팩션 사이클당 한 번 실행됩니다 (`sessions.json`에서 추적).
- 플러시는 임베디드 Pi 세션에서만 실행됩니다 (CLI 백엔드는 건너뜀).
- 세션 워크스페이스가 읽기 전용인 경우 (`workspaceAccess: "ro"` 또는 `"none"`) 플러시를 건너뜁니다.
- 워크스페이스 파일 레이아웃과 쓰기 패턴은 [메모리](/concepts/memory)를 참조하세요.

Pi는 확장 API에 `session_before_compact` 훅도 노출하지만, OpenClaw의
플러시 로직은 현재 게이트웨이 측에 있습니다.

---

## 트러블슈팅 체크리스트

- 세션 키가 잘못되었나요? [/concepts/session](/concepts/session)에서 시작하여 `/status`의 `sessionKey`를 확인하세요.
- 저장소와 트랜스크립트 불일치? 게이트웨이 호스트와 `openclaw status`의 저장소 경로를 확인하세요.
- 컴팩션 스팸? 확인사항:
  - 모델 컨텍스트 윈도우 (너무 작음)
  - 컴팩션 설정 (모델 윈도우에 비해 `reserveTokens`가 너무 높으면 조기 컴팩션 발생 가능)
  - 도구 결과 비대: 세션 프루닝 활성화/튜닝
- 사일런트 턴 누출? 응답이 `NO_REPLY` (정확한 토큰)로 시작하는지 확인하고 스트리밍 억제 수정이 포함된 빌드인지 확인하세요.
