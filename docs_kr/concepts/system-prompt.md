---
summary: "OpenClaw 시스템 프롬프트의 구성 요소와 조립 방법"
read_when:
  - Editing system prompt text, tools list, or time/heartbeat sections
  - Changing workspace bootstrap or skills injection behavior
title: "시스템 프롬프트"
---

# 시스템 프롬프트

OpenClaw는 모든 에이전트 실행에 대해 커스텀 시스템 프롬프트를 구성합니다. 이 프롬프트는 **OpenClaw 소유**이며 pi-coding-agent 기본 프롬프트를 사용하지 않습니다.

프롬프트는 OpenClaw에 의해 조립되고 각 에이전트 실행에 주입됩니다.

## 구조

프롬프트는 의도적으로 간결하며 고정 섹션을 사용합니다:

- **Tooling**: 현재 도구 목록 + 짧은 설명.
- **Safety**: 권한 추구 행동이나 감독 우회를 방지하기 위한 짧은 가드레일 알림.
- **Skills** (사용 가능한 경우): 모델에게 요청 시 스킬 지침을 로드하는 방법을 알려줍니다.
- **OpenClaw Self-Update**: `config.apply`와 `update.run`을 실행하는 방법.
- **Workspace**: 작업 디렉토리 (`agents.defaults.workspace`).
- **Documentation**: OpenClaw 문서(저장소 또는 npm 패키지)의 로컬 경로와 참조 시기.
- **Workspace Files (injected)**: 부트스트랩 파일이 아래에 포함되어 있음을 나타냅니다.
- **Sandbox** (활성화된 경우): 샌드박스 런타임, 샌드박스 경로, elevated exec 사용 가능 여부를 나타냅니다.
- **Current Date & Time**: 사용자 로컬 시간, 시간대 및 시간 형식.
- **Reply Tags**: 지원되는 제공자를 위한 선택적 응답 태그 구문.
- **Heartbeats**: 하트비트 프롬프트 및 확인 동작.
- **Runtime**: 호스트, OS, node, 모델, repo 루트(감지된 경우), thinking level (한 줄).
- **Reasoning**: 현재 가시성 수준 + /reasoning 토글 힌트.

시스템 프롬프트의 안전 가드레일은 권고적입니다. 모델 동작을 안내하지만 정책을 시행하지는 않습니다. 하드 시행을 위해 도구 정책, exec 승인, 샌드박싱, 채널 허용 목록을 사용하세요; 운영자는 설계에 의해 이를 비활성화할 수 있습니다.

## 프롬프트 모드

OpenClaw는 서브 에이전트를 위해 더 작은 시스템 프롬프트를 렌더링할 수 있습니다. 런타임은 각 실행에 대해 `promptMode`를 설정합니다(사용자 대면 구성이 아님):

- `full` (기본값): 위의 모든 섹션을 포함합니다.
- `minimal`: 서브 에이전트에 사용됩니다; **Skills**, **Memory Recall**, **OpenClaw
  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies**, **Heartbeats**를 생략합니다. Tooling, **Safety**,
  Workspace, Sandbox, Current Date & Time(알려진 경우), Runtime, 주입된 컨텍스트는 계속 사용할 수 있습니다.
- `none`: 기본 식별 줄만 반환합니다.

`promptMode=minimal`일 때, 추가 주입된 프롬프트는 **Group Chat Context** 대신 **Subagent
Context**로 라벨링됩니다.

## 워크스페이스 부트스트랩 주입

부트스트랩 파일은 트리밍되어 **Project Context** 아래에 추가되므로 모델이 명시적 읽기 없이도 식별 및 프로필 컨텍스트를 볼 수 있습니다:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (새 워크스페이스에서만)
- `MEMORY.md` 및/또는 `memory.md` (워크스페이스에 있는 경우; 하나 또는 둘 다 주입될 수 있음)

이 모든 파일은 매 턴마다 **컨텍스트 윈도우에 주입**되므로 토큰을 소비합니다. 간결하게 유지하세요 - 특히 시간이 지남에 따라 커질 수 있어 예상치 못하게 높은 컨텍스트 사용량과 더 빈번한 압축으로 이어질 수 있는 `MEMORY.md`는 더욱 그렇습니다.

> **참고:** `memory/*.md` 일일 파일은 자동으로 주입되지 **않습니다**. `memory_search` 및 `memory_get` 도구를 통해 요청 시 접근하므로, 모델이 명시적으로 읽지 않는 한 컨텍스트 윈도우에 포함되지 않습니다.

큰 파일은 마커와 함께 잘립니다. 파일당 최대 크기는
`agents.defaults.bootstrapMaxChars`(기본값: 20000)로 제어됩니다. 파일 전반에 걸쳐 주입된 부트스트랩
콘텐츠의 총량은 `agents.defaults.bootstrapTotalMaxChars`(기본값: 150000)로 제한됩니다. 누락된 파일은 짧은 누락 파일 마커를 주입합니다.

서브 에이전트 세션은 `AGENTS.md`와 `TOOLS.md`만 주입합니다(서브 에이전트 컨텍스트를 작게 유지하기 위해 다른 부트스트랩 파일은 필터링됩니다).

내부 후크는 `agent:bootstrap`를 통해 이 단계를 가로채서 주입된 부트스트랩 파일을 변경하거나 교체할 수 있습니다(예: 대체 페르소나를 위해 `SOUL.md`를 교체).

주입된 각 파일의 기여도(원본 vs 주입, 잘림, 도구 스키마 오버헤드 포함)를 검사하려면 `/context list` 또는 `/context detail`을 사용하세요. [Context](/concepts/context)를 참조하세요.

## 시간 처리

시스템 프롬프트에는 사용자 시간대가 알려진 경우 전용 **Current Date & Time** 섹션이 포함됩니다. 프롬프트 캐시를 안정적으로 유지하기 위해, 이제 **시간대**만 포함합니다(동적 시계나 시간 형식 없음).

에이전트가 현재 시간이 필요할 때 `session_status`를 사용하세요; 상태 카드에 타임스탬프 줄이 포함됩니다.

다음으로 구성하세요:

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

전체 동작 세부 사항은 [Date & Time](/date-time)을 참조하세요.

## 스킬

적격 스킬이 있으면, OpenClaw는 각 스킬의 **파일 경로**를 포함하는 간결한 **사용 가능한 스킬 목록** (`formatSkillsForPrompt`)을 주입합니다. 프롬프트는 나열된 위치(워크스페이스, 관리형 또는 번들)에서 `read`를 사용하여 SKILL.md를 로드하도록 모델에 지시합니다. 적격 스킬이 없으면 Skills 섹션이 생략됩니다.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

이를 통해 기본 프롬프트를 작게 유지하면서도 대상 지정 스킬 사용이 가능합니다.

## 문서

사용 가능한 경우, 시스템 프롬프트에는 로컬 OpenClaw 문서 디렉토리(저장소 워크스페이스의 `docs/` 또는 번들된 npm 패키지 문서)를 가리키는 **Documentation** 섹션이 포함됩니다. 또한 공개 미러, 소스 저장소, 커뮤니티 Discord, 그리고 스킬 검색을 위한 ClawHub([https://clawhub.com](https://clawhub.com))도 언급합니다. 프롬프트는 OpenClaw 동작, 명령, 구성 또는 아키텍처에 대해 먼저 로컬 문서를 참조하도록 모델에 지시하고, 가능한 경우 `openclaw status`를 직접 실행하도록 합니다(접근 권한이 없을 때만 사용자에게 요청).
