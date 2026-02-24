---
summary: "서브 에이전트: 요청자 채팅에 결과를 다시 알리는 격리된 에이전트 실행 생성"
read_when:
  - 에이전트를 통한 백그라운드/병렬 작업이 필요한 경우
  - sessions_spawn 또는 서브 에이전트 도구 정책을 변경하는 경우
  - 스레드 바인딩된 서브에이전트 세션을 구현하거나 문제를 해결하는 경우
title: "서브 에이전트"
---

# 서브 에이전트

서브 에이전트는 기존 에이전트 실행에서 생성된 백그라운드 에이전트 실행입니다. 자체 세션(`agent:<agentId>:subagent:<uuid>`)에서 실행되며, 완료되면 요청자 채팅 채널로 결과를 **알립니다**.

## 슬래시 명령

`/subagents`를 사용하여 **현재 세션**의 서브 에이전트 실행을 검사하거나 제어합니다:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`
- `/subagents steer <id|#> <message>`
- `/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]`

스레드 바인딩 제어:

이 명령은 영구 스레드 바인딩을 지원하는 채널에서 작동합니다. 아래의 **스레드 지원 채널**을 참조하세요.

- `/focus <subagent-label|session-key|session-id|session-label>`
- `/unfocus`
- `/agents`
- `/session ttl <duration|off>`

`/subagents info`는 실행 메타데이터(상태, 타임스탬프, 세션 id, 트랜스크립트 경로, 정리)를 표시합니다.

### 생성 동작

`/subagents spawn`은 내부 릴레이가 아닌 사용자 명령으로 백그라운드 서브 에이전트를 시작하며, 실행이 완료되면 요청자 채팅에 하나의 최종 완료 업데이트를 보냅니다.

- spawn 명령은 논블로킹입니다; 즉시 실행 id를 반환합니다.
- 완료 시 서브 에이전트는 요청자 채팅 채널에 요약/결과 메시지를 알립니다.
- 수동 생성의 경우 전달은 복원력이 있습니다:
  - OpenClaw는 안정적인 멱등성 키로 직접 `agent` 전달을 먼저 시도합니다.
  - 직접 전달이 실패하면 큐 라우팅으로 폴백합니다.
  - 큐 라우팅도 사용할 수 없으면 짧은 지수 백오프로 알림을 재시도한 후 최종 포기합니다.
- 완료 메시지는 시스템 메시지이며 다음을 포함합니다:
  - `Result`(assistant 응답 텍스트, 또는 assistant 응답이 비어 있으면 최신 `toolResult`)
  - `Status`(`completed successfully` / `failed` / `timed out`)
  - 간결한 런타임/토큰 통계
- `--model`과 `--thinking`은 해당 특정 실행의 기본값을 재정의합니다.
- 완료 후 세부 사항과 출력을 검사하려면 `info`/`log`를 사용하세요.
- `/subagents spawn`은 일회성 모드(`mode: "run"`)입니다. 영구 스레드 바인딩 세션에는 `thread: true`와 `mode: "session"`이 포함된 `sessions_spawn`을 사용하세요.

주요 목표:

- 메인 실행을 차단하지 않고 "리서치 / 긴 작업 / 느린 도구" 작업을 병렬화합니다.
- 기본적으로 서브 에이전트를 격리합니다(세션 분리 + 선택적 샌드박싱).
- 도구 표면을 오용하기 어렵게 유지: 서브 에이전트는 기본적으로 세션 도구를 받지 **않습니다**.
- 오케스트레이터 패턴을 위한 구성 가능한 중첩 깊이를 지원합니다.

비용 참고: 각 서브 에이전트는 **자체** 컨텍스트와 토큰 사용량을 가집니다. 무거운 또는 반복적인 작업의 경우 서브 에이전트에 저렴한 모델을 설정하고 메인 에이전트는 더 높은 품질의 모델로 유지하세요.
`agents.defaults.subagents.model` 또는 에이전트별 재정의를 통해 구성할 수 있습니다.

## 도구

`sessions_spawn`을 사용합니다:

- 서브 에이전트 실행을 시작합니다(`deliver: false`, 전역 레인: `subagent`)
- 그런 다음 알림 단계를 실행하고 요청자 채팅 채널에 알림 응답을 게시합니다
- 기본 모델: `agents.defaults.subagents.model`(또는 에이전트별 `agents.list[].subagents.model`)을 설정하지 않으면 호출자를 상속합니다; 명시적 `sessions_spawn.model`이 여전히 우선합니다.
- 기본 thinking: `agents.defaults.subagents.thinking`(또는 에이전트별 `agents.list[].subagents.thinking`)을 설정하지 않으면 호출자를 상속합니다; 명시적 `sessions_spawn.thinking`이 여전히 우선합니다.

도구 파라미터:

- `task`(필수)
- `label?`(선택 사항)
- `agentId?`(선택 사항; 허용되면 다른 에이전트 id로 생성)
- `model?`(선택 사항; 서브 에이전트 모델을 재정의; 유효하지 않은 값은 건너뛰고 서브 에이전트는 도구 결과에 경고와 함께 기본 모델로 실행)
- `thinking?`(선택 사항; 서브 에이전트 실행의 thinking 수준을 재정의)
- `runTimeoutSeconds?`(기본값 `0`; 설정하면 N초 후 서브 에이전트 실행이 중단)
- `thread?`(기본값 `false`; `true`이면 이 서브 에이전트 세션에 대한 채널 스레드 바인딩을 요청)
- `mode?`(`run|session`)
  - 기본값은 `run`
  - `thread: true`이고 `mode`가 생략되면 기본값이 `session`이 됨
  - `mode: "session"`은 `thread: true`가 필요
- `cleanup?`(`delete|keep`, 기본값 `keep`)

## 스레드 바인딩 세션

채널에서 스레드 바인딩이 활성화되면 서브 에이전트가 스레드에 바인딩되어 해당 스레드의 후속 사용자 메시지가 동일한 서브 에이전트 세션으로 계속 라우팅될 수 있습니다.

### 스레드 지원 채널

- Discord(현재 유일하게 지원되는 채널): 영구 스레드 바인딩 서브에이전트 세션(`thread: true`가 포함된 `sessions_spawn`), 수동 스레드 제어(`/focus`, `/unfocus`, `/agents`, `/session ttl`), 어댑터 키 `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.ttlHours`, `channels.discord.threadBindings.spawnSubagentSessions`를 지원합니다.

빠른 흐름:

1. `thread: true`(그리고 선택적으로 `mode: "session"`)를 사용하여 `sessions_spawn`으로 생성합니다.
2. OpenClaw는 활성 채널에서 해당 세션 대상에 스레드를 생성하거나 바인딩합니다.
3. 해당 스레드의 응답 및 후속 메시지가 바인딩된 세션으로 라우팅됩니다.
4. `/session ttl`을 사용하여 자동 언포커스 TTL을 검사/업데이트합니다.
5. 수동으로 분리하려면 `/unfocus`를 사용합니다.

수동 제어:

- `/focus <target>`는 현재 스레드(또는 새 스레드)를 서브 에이전트/세션 대상에 바인딩합니다.
- `/unfocus`는 현재 바인딩된 스레드의 바인딩을 제거합니다.
- `/agents`는 활성 실행과 바인딩 상태(`thread:<id>` 또는 `unbound`)를 나열합니다.
- `/session ttl`은 포커스된 바인딩 스레드에서만 작동합니다.

구성 스위치:

- 전역 기본값: `session.threadBindings.enabled`, `session.threadBindings.ttlHours`
- 채널 재정의 및 생성 자동 바인딩 키는 어댑터별입니다. 위의 **스레드 지원 채널**을 참조하세요.

현재 어댑터 세부 사항은 [구성 참조](/gateway/configuration-reference) 및 [슬래시 명령](/tools/slash-commands)을 참조하세요.

허용 목록:

- `agents.list[].subagents.allowAgents`: `agentId`를 통해 대상으로 지정할 수 있는 에이전트 id 목록(`["*"]`로 모든 것을 허용). 기본값: 요청자 에이전트만.

검색:

- `agents_list`를 사용하여 현재 `sessions_spawn`에 허용된 에이전트 id를 확인합니다.

자동 아카이브:

- 서브 에이전트 세션은 `agents.defaults.subagents.archiveAfterMinutes`(기본값: 60) 후 자동으로 아카이브됩니다.
- 아카이브는 `sessions.delete`를 사용하고 트랜스크립트를 `*.deleted.<timestamp>`(동일 폴더)로 이름 변경합니다.
- `cleanup: "delete"`는 알림 직후 아카이브합니다(이름 변경을 통해 트랜스크립트는 여전히 유지).
- 자동 아카이브는 최선의 노력입니다; 게이트웨이가 재시작되면 보류 중인 타이머가 손실됩니다.
- `runTimeoutSeconds`는 자동 아카이브하지 않습니다; 실행만 중지합니다. 세션은 자동 아카이브까지 유지됩니다.
- 자동 아카이브는 depth-1 및 depth-2 세션에 동등하게 적용됩니다.

## 중첩 서브 에이전트

기본적으로 서브 에이전트는 자체 서브 에이전트를 생성할 수 없습니다(`maxSpawnDepth: 1`). `maxSpawnDepth: 2`로 설정하여 한 수준의 중첩을 활성화할 수 있으며, 이는 **오케스트레이터 패턴**을 허용합니다: 메인 → 오케스트레이터 서브 에이전트 → 워커 서브서브 에이전트.

### 활성화 방법

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // 서브 에이전트가 자식을 생성할 수 있도록 허용 (기본값: 1)
        maxChildrenPerAgent: 5, // 에이전트 세션당 최대 활성 자식 수 (기본값: 5)
        maxConcurrent: 8, // 전역 동시성 레인 상한 (기본값: 8)
      },
    },
  },
}
```

### 깊이 수준

| 깊이 | 세션 키 형태                                  | 역할                                          | 생성 가능?                   |
| ---- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0    | `agent:<id>:main`                            | 메인 에이전트                                  | 항상                         |
| 1    | `agent:<id>:subagent:<uuid>`                 | 서브 에이전트(depth 2 허용 시 오케스트레이터)   | `maxSpawnDepth >= 2`인 경우만 |
| 2    | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | 서브서브 에이전트(리프 워커)                    | 절대 불가                    |

### 알림 체인

결과는 체인을 따라 위로 흐릅니다:

1. Depth-2 워커 완료 → 부모(depth-1 오케스트레이터)에 알림
2. Depth-1 오케스트레이터가 알림을 받고 결과를 종합, 완료 → 메인에 알림
3. 메인 에이전트가 알림을 받고 사용자에게 전달

각 수준은 직접 자식의 알림만 봅니다.

### 깊이별 도구 정책

- **Depth 1(오케스트레이터, `maxSpawnDepth >= 2`일 때)**: `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`를 받아 자식을 관리할 수 있습니다. 다른 세션/시스템 도구는 거부됩니다.
- **Depth 1(리프, `maxSpawnDepth == 1`일 때)**: 세션 도구 없음(현재 기본 동작).
- **Depth 2(리프 워커)**: 세션 도구 없음 -- `sessions_spawn`은 depth 2에서 항상 거부됩니다. 추가 자식을 생성할 수 없습니다.

### 에이전트별 생성 제한

각 에이전트 세션(어떤 깊이에서든)은 한 번에 최대 `maxChildrenPerAgent`(기본값: 5)개의 활성 자식을 가질 수 있습니다. 이는 단일 오케스트레이터에서의 폭주 팬아웃을 방지합니다.

### 연쇄 중지

depth-1 오케스트레이터를 중지하면 자동으로 모든 depth-2 자식이 중지됩니다:

- 메인 채팅에서 `/stop`은 모든 depth-1 에이전트를 중지하고 depth-2 자식으로 연쇄합니다.
- `/subagents kill <id>`는 특정 서브 에이전트를 중지하고 자식으로 연쇄합니다.
- `/subagents kill all`은 요청자의 모든 서브 에이전트를 중지하고 연쇄합니다.

## 인증

서브 에이전트 인증은 세션 유형이 아닌 **에이전트 id**로 해석됩니다:

- 서브 에이전트 세션 키는 `agent:<agentId>:subagent:<uuid>`입니다.
- 인증 저장소는 해당 에이전트의 `agentDir`에서 로드됩니다.
- 메인 에이전트의 인증 프로필은 **폴백**으로 병합됩니다; 충돌 시 에이전트 프로필이 메인 프로필을 재정의합니다.

참고: 병합은 추가적이므로 메인 프로필은 항상 폴백으로 사용 가능합니다. 에이전트별 완전 격리 인증은 아직 지원되지 않습니다.

## 알림

서브 에이전트는 알림 단계를 통해 결과를 보고합니다:

- 알림 단계는 서브 에이전트 세션(요청자 세션이 아님)에서 실행됩니다.
- 서브 에이전트가 정확히 `ANNOUNCE_SKIP`으로 응답하면 아무것도 게시되지 않습니다.
- 그렇지 않으면 알림 응답이 후속 `agent` 호출(`deliver=true`)을 통해 요청자 채팅 채널에 게시됩니다.
- 알림 응답은 채널 어댑터에서 사용 가능한 경우 스레드/토픽 라우팅을 보존합니다.
- 알림 메시지는 안정적인 템플릿으로 정규화됩니다:
  - `Status:` 실행 결과에서 파생(`success`, `error`, `timeout` 또는 `unknown`).
  - `Result:` 알림 단계의 요약 콘텐츠(또는 누락 시 `(not available)`).
  - `Notes:` 오류 세부 사항 및 기타 유용한 컨텍스트.
- `Status`는 모델 출력에서 추론되지 않습니다; 런타임 결과 신호에서 옵니다.

알림 페이로드는 끝에 통계 행을 포함합니다(래핑된 경우에도):

- 런타임(예: `runtime 5m12s`)
- 토큰 사용량(입력/출력/총계)
- 모델 가격이 구성된 경우(`models.providers.*.models[].cost`) 예상 비용
- `sessionKey`, `sessionId` 및 트랜스크립트 경로(메인 에이전트가 `sessions_history`를 통해 기록을 가져오거나 디스크의 파일을 검사할 수 있도록)

## 도구 정책(서브 에이전트 도구)

기본적으로 서브 에이전트는 세션 도구 및 시스템 도구를 **제외한 모든 도구**를 받습니다:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2`이면 depth-1 오케스트레이터 서브 에이전트는 자식을 관리할 수 있도록 `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`를 추가로 받습니다.

구성을 통한 재정의:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny가 우선
        deny: ["gateway", "cron"],
        // allow가 설정되면 허용 전용이 됩니다(deny가 여전히 우선)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## 동시성

서브 에이전트는 전용 인프로세스 큐 레인을 사용합니다:

- 레인 이름: `subagent`
- 동시성: `agents.defaults.subagents.maxConcurrent`(기본값 `8`)

## 중지

- 요청자 채팅에서 `/stop`을 보내면 요청자 세션을 중단하고 해당 세션에서 생성된 모든 활성 서브 에이전트 실행을 중지하며 중첩 자식으로 연쇄합니다.
- `/subagents kill <id>`는 특정 서브 에이전트를 중지하고 자식으로 연쇄합니다.

## 제한 사항

- 서브 에이전트 알림은 **최선의 노력**입니다. 게이트웨이가 재시작되면 보류 중인 "알림 반환" 작업이 손실됩니다.
- 서브 에이전트는 여전히 동일한 게이트웨이 프로세스 리소스를 공유합니다; `maxConcurrent`를 안전 밸브로 취급하세요.
- `sessions_spawn`은 항상 논블로킹입니다: 즉시 `{ status: "accepted", runId, childSessionKey }`를 반환합니다.
- 서브 에이전트 컨텍스트는 `AGENTS.md` + `TOOLS.md`만 주입합니다(`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` 또는 `BOOTSTRAP.md` 없음).
- 최대 중첩 깊이는 5(`maxSpawnDepth` 범위: 1-5)입니다. 대부분의 사용 사례에는 Depth 2가 권장됩니다.
- `maxChildrenPerAgent`는 세션당 활성 자식을 제한합니다(기본값: 5, 범위: 1-20).
