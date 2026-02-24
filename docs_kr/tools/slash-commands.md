---
summary: "슬래시 명령: 텍스트 vs 네이티브, 구성 및 지원되는 명령"
read_when:
  - 채팅 명령을 사용하거나 구성하는 경우
  - 명령 라우팅 또는 권한을 디버깅하는 경우
title: "슬래시 명령"
---

# 슬래시 명령

명령은 Gateway에서 처리됩니다. 대부분의 명령은 `/`로 시작하는 **독립 실행형** 메시지로 보내야 합니다.
호스트 전용 bash 채팅 명령은 `! <cmd>`를 사용합니다(`/bash <cmd>`가 별칭).

두 가지 관련 시스템이 있습니다:

- **명령**: 독립 실행형 `/...` 메시지.
- **지시문**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  - 지시문은 모델이 보기 전에 메시지에서 제거됩니다.
  - 일반 채팅 메시지(지시문만 있는 것이 아닌)에서는 "인라인 힌트"로 취급되며 세션 설정을 **유지하지 않습니다**.
  - 지시문만 있는 메시지(메시지에 지시문만 포함)에서는 세션에 유지되고 확인 응답이 전송됩니다.
  - 지시문은 **인증된 발신자**에게만 적용됩니다. `commands.allowFrom`이 설정되면 명령 및 지시문에 사용되는 유일한 허용 목록입니다; 그렇지 않으면 채널 허용 목록/페어링 및 `commands.useAccessGroups`에서 인증이 제공됩니다.
    인증되지 않은 발신자는 지시문이 일반 텍스트로 취급됩니다.

또한 몇 가지 **인라인 단축키**(허용 목록/인증된 발신자만)가 있습니다: `/help`, `/commands`, `/status`, `/whoami`(`/id`).
이들은 즉시 실행되고 모델이 메시지를 보기 전에 제거되며 나머지 텍스트는 정상 흐름을 통해 계속됩니다.

## 구성

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

- `commands.text`(기본값 `true`)는 채팅 메시지에서 `/...` 파싱을 활성화합니다.
  - 네이티브 명령이 없는 표면(WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams)에서는 이를 `false`로 설정해도 텍스트 명령이 여전히 작동합니다.
- `commands.native`(기본값 `"auto"`)는 네이티브 명령을 등록합니다.
  - Auto: Discord/Telegram에서 활성화; Slack에서 비활성화(슬래시 명령을 추가할 때까지); 네이티브 지원이 없는 프로바이더에서는 무시.
  - 프로바이더별로 재정의하려면 `channels.discord.commands.native`, `channels.telegram.commands.native` 또는 `channels.slack.commands.native`를 설정하세요(bool 또는 `"auto"`).
  - `false`는 시작 시 Discord/Telegram에서 이전에 등록된 명령을 지웁니다. Slack 명령은 Slack 앱에서 관리되며 자동으로 제거되지 않습니다.
- `commands.nativeSkills`(기본값 `"auto"`)는 지원되는 경우 **스킬** 명령을 네이티브로 등록합니다.
  - Auto: Discord/Telegram에서 활성화; Slack에서 비활성화(Slack은 스킬당 슬래시 명령을 생성해야 함).
  - 프로바이더별로 재정의하려면 `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills` 또는 `channels.slack.commands.nativeSkills`를 설정하세요(bool 또는 `"auto"`).
- `commands.bash`(기본값 `false`)는 `! <cmd>`로 호스트 셸 명령을 실행할 수 있게 합니다(`/bash <cmd>`가 별칭; `tools.elevated` 허용 목록 필요).
- `commands.bashForegroundMs`(기본값 `2000`)는 bash가 백그라운드 모드로 전환하기 전 대기 시간을 제어합니다(`0`은 즉시 백그라운드).
- `commands.config`(기본값 `false`)는 `/config`를 활성화합니다(`openclaw.json` 읽기/쓰기).
- `commands.debug`(기본값 `false`)는 `/debug`를 활성화합니다(런타임 전용 재정의).
- `commands.allowFrom`(선택 사항)은 명령 인증을 위한 프로바이더별 허용 목록을 설정합니다. 구성된 경우 명령 및 지시문의 유일한 인증 소스가 됩니다(채널 허용 목록/페어링 및 `commands.useAccessGroups`는 무시됨). 전역 기본값에 `"*"`를 사용하세요; 프로바이더별 키가 이를 재정의합니다.
- `commands.useAccessGroups`(기본값 `true`)는 `commands.allowFrom`이 설정되지 않은 경우 명령에 대한 허용 목록/정책을 적용합니다.

## 명령 목록

텍스트 + 네이티브(활성화 시):

- `/help`
- `/commands`
- `/skill <name> [input]`(이름으로 스킬 실행)
- `/status`(현재 상태 표시; 사용 가능한 경우 현재 모델 프로바이더의 프로바이더 사용량/쿼터 포함)
- `/allowlist`(허용 목록 항목 나열/추가/제거)
- `/approve <id> allow-once|allow-always|deny`(exec 승인 프롬프트 해결)
- `/context [list|detail|json]`("컨텍스트" 설명; `detail`은 파일별 + 도구별 + 스킬별 + 시스템 프롬프트 크기 표시)
- `/export-session [path]`(별칭: `/export`)(현재 세션을 전체 시스템 프롬프트와 함께 HTML로 내보내기)
- `/whoami`(발신자 id 표시; 별칭: `/id`)
- `/session ttl <duration|off>`(TTL 등 세션 수준 설정 관리)
- `/subagents list|kill|log|info|send|steer|spawn`(현재 세션의 서브 에이전트 실행을 검사, 제어 또는 생성)
- `/agents`(이 세션의 스레드 바인딩된 에이전트 나열)
- `/focus <target>`(Discord: 이 스레드 또는 새 스레드를 세션/서브에이전트 대상에 바인딩)
- `/unfocus`(Discord: 현재 스레드 바인딩 제거)
- `/kill <id|#|all>`(이 세션에서 실행 중인 하나 또는 모든 서브 에이전트를 즉시 중단; 확인 메시지 없음)
- `/steer <id|#> <message>`(실행 중인 서브 에이전트를 즉시 조종: 가능하면 실행 중에, 그렇지 않으면 현재 작업을 중단하고 조종 메시지로 재시작)
- `/tell <id|#> <message>`(`/steer`의 별칭)
- `/config show|get|set|unset`(디스크에 구성 유지, 소유자 전용; `commands.config: true` 필요)
- `/debug show|set|unset|reset`(런타임 재정의, 소유자 전용; `commands.debug: true` 필요)
- `/usage off|tokens|full|cost`(응답별 사용량 푸터 또는 로컬 비용 요약)
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`(TTS 제어; [/tts](/tts) 참조)
  - Discord: 네이티브 명령은 `/voice`(Discord가 `/tts`를 예약); 텍스트 `/tts`는 여전히 작동.
- `/stop`
- `/restart`
- `/dock-telegram`(별칭: `/dock_telegram`)(답장을 Telegram으로 전환)
- `/dock-discord`(별칭: `/dock_discord`)(답장을 Discord로 전환)
- `/dock-slack`(별칭: `/dock_slack`)(답장을 Slack으로 전환)
- `/activation mention|always`(그룹 전용)
- `/send on|off|inherit`(소유자 전용)
- `/reset` 또는 `/new [model]`(선택적 모델 힌트; 나머지는 통과)
- `/think <off|minimal|low|medium|high|xhigh>`(모델/프로바이더별 동적 선택; 별칭: `/thinking`, `/t`)
- `/verbose on|full|off`(별칭: `/v`)
- `/reasoning on|off|stream`(별칭: `/reason`; 활성화 시 `Reasoning:` 접두사가 있는 별도 메시지 전송; `stream` = Telegram 초안만)
- `/elevated on|off|ask|full`(별칭: `/elev`; `full`은 exec 승인을 건너뜀)
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`(`/exec`를 보내면 현재 표시)
- `/model <name>`(별칭: `/models`; 또는 `agents.defaults.models.*.alias`의 `/<alias>`)
- `/queue <mode>`(`debounce:2s cap:25 drop:summarize` 같은 옵션 추가; `/queue`를 보내면 현재 설정 표시)
- `/bash <command>`(호스트 전용; `! <command>`의 별칭; `commands.bash: true` + `tools.elevated` 허용 목록 필요)

텍스트 전용:

- `/compact [instructions]`([/concepts/compaction](/concepts/compaction) 참조)
- `! <command>`(호스트 전용; 한 번에 하나씩; 장시간 실행 작업에는 `!poll` + `!stop` 사용)
- `!poll`(출력/상태 확인; 선택적 `sessionId` 허용; `/bash poll`도 작동)
- `!stop`(실행 중인 bash 작업 중지; 선택적 `sessionId` 허용; `/bash stop`도 작동)

참고:

- 명령은 명령과 인수 사이에 선택적 `:`를 허용합니다(예: `/think: high`, `/send: on`, `/help:`).
- `/new <model>`은 모델 별칭, `provider/model` 또는 프로바이더 이름(퍼지 매치)을 허용합니다; 일치하는 것이 없으면 텍스트가 메시지 본문으로 취급됩니다.
- 전체 프로바이더 사용량 분석은 `openclaw status --usage`를 사용하세요.
- `/allowlist add|remove`는 `commands.config=true`가 필요하며 채널 `configWrites`를 따릅니다.
- `/usage`는 응답별 사용량 푸터를 제어합니다; `/usage cost`는 OpenClaw 세션 로그의 로컬 비용 요약을 출력합니다.
- `/restart`는 기본적으로 활성화됩니다; 비활성화하려면 `commands.restart: false`를 설정하세요.
- Discord 전용 네이티브 명령: `/vc join|leave|status`는 음성 채널을 제어합니다(`channels.discord.voice` 및 네이티브 명령 필요; 텍스트로는 사용 불가).
- Discord 스레드 바인딩 명령(`/focus`, `/unfocus`, `/agents`, `/session ttl`)은 유효한 스레드 바인딩이 활성화되어야 합니다(`session.threadBindings.enabled` 및/또는 `channels.discord.threadBindings.enabled`).
- `/verbose`는 디버깅 및 추가 가시성을 위한 것입니다; 일반 사용에서는 **꺼둔 상태**를 유지하세요.
- 도구 실패 요약은 관련이 있을 때 여전히 표시되지만 상세한 실패 텍스트는 `/verbose`가 `on` 또는 `full`일 때만 포함됩니다.
- `/reasoning`(그리고 `/verbose`)은 그룹 설정에서 위험합니다: 의도하지 않은 내부 추론이나 도구 출력이 노출될 수 있습니다. 특히 그룹 채팅에서는 꺼두는 것을 선호하세요.
- **빠른 경로:** 허용 목록 발신자의 명령만 있는 메시지는 즉시 처리됩니다(큐 + 모델 우회).
- **그룹 멘션 게이팅:** 허용 목록 발신자의 명령만 있는 메시지는 멘션 요구 사항을 우회합니다.
- **인라인 단축키(허용 목록 발신자만):** 특정 명령은 일반 메시지에 포함되어 작동하며 모델이 나머지 텍스트를 보기 전에 제거됩니다.
  - 예: `hey /status`는 상태 응답을 트리거하고 나머지 텍스트는 정상 흐름을 통해 계속됩니다.
- 현재: `/help`, `/commands`, `/status`, `/whoami`(`/id`).
- 인증되지 않은 명령만 있는 메시지는 자동으로 무시되며 인라인 `/...` 토큰은 일반 텍스트로 취급됩니다.
- **스킬 명령:** `user-invocable` 스킬은 슬래시 명령으로 노출됩니다. 이름은 `a-z0-9_`(최대 32자)로 정리됩니다; 충돌 시 숫자 접미사가 붙습니다(예: `_2`).
  - `/skill <name> [input]`은 이름으로 스킬을 실행합니다(네이티브 명령 제한으로 스킬별 명령이 불가능할 때 유용).
  - 기본적으로 스킬 명령은 일반 요청으로 모델에 전달됩니다.
  - 스킬은 선택적으로 `command-dispatch: tool`을 선언하여 명령을 도구로 직접 라우팅할 수 있습니다(결정적, 모델 없음).
  - 예: `/prose`(OpenProse 플러그인) -- [OpenProse](/prose) 참조.
- **네이티브 명령 인수:** Discord는 동적 옵션에 자동 완성을 사용합니다(필수 인수를 생략하면 버튼 메뉴). Telegram과 Slack은 명령이 선택지를 지원하고 인수를 생략하면 버튼 메뉴를 표시합니다.

## 사용 표면(어디에 무엇이 표시되는지)

- **프로바이더 사용량/쿼터**(예: "Claude 80% left")는 사용량 추적이 활성화된 경우 현재 모델 프로바이더의 `/status`에 표시됩니다.
- **응답별 토큰/비용**은 `/usage off|tokens|full`로 제어됩니다(일반 응답에 추가).
- `/model status`는 사용량이 아닌 **모델/인증/엔드포인트**에 관한 것입니다.

## 모델 선택(`/model`)

`/model`은 지시문으로 구현됩니다.

예제:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

참고:

- `/model`과 `/model list`는 간결한 번호 선택기를 표시합니다(모델 패밀리 + 사용 가능한 프로바이더).
- Discord에서 `/model`과 `/models`는 프로바이더 및 모델 드롭다운과 Submit 단계가 있는 대화형 선택기를 엽니다.
- `/model <#>`는 해당 선택기에서 선택합니다(가능하면 현재 프로바이더를 선호).
- `/model status`는 구성된 프로바이더 엔드포인트(`baseUrl`)와 API 모드(`api`)를 포함한 상세 보기를 표시합니다.

## 디버그 재정의

`/debug`는 **런타임 전용** 구성 재정의를 설정할 수 있습니다(메모리, 디스크 아님). 소유자 전용. 기본 비활성화; `commands.debug: true`로 활성화하세요.

예제:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

참고:

- 재정의는 새 구성 읽기에 즉시 적용되지만 `openclaw.json`에 **쓰지 않습니다**.
- 모든 재정의를 지우고 디스크 구성으로 돌아가려면 `/debug reset`을 사용하세요.

## 구성 업데이트

`/config`는 디스크 구성(`openclaw.json`)에 씁니다. 소유자 전용. 기본 비활성화; `commands.config: true`로 활성화하세요.

예제:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

참고:

- 구성은 쓰기 전에 유효성 검사됩니다; 유효하지 않은 변경은 거부됩니다.
- `/config` 업데이트는 재시작 후에도 유지됩니다.

## 표면 참고 사항

- **텍스트 명령**은 일반 채팅 세션에서 실행됩니다(DM은 `main`을 공유, 그룹은 자체 세션 보유).
- **네이티브 명령**은 격리된 세션을 사용합니다:
  - Discord: `agent:<agentId>:discord:slash:<userId>`
  - Slack: `agent:<agentId>:slack:slash:<userId>`(`channels.slack.slashCommand.sessionPrefix`를 통해 접두사 구성 가능)
  - Telegram: `telegram:slash:<userId>`(`CommandTargetSessionKey`를 통해 채팅 세션 대상)
- **`/stop`**은 활성 채팅 세션을 대상으로 하여 현재 실행을 중단할 수 있습니다.
- **Slack:** `channels.slack.slashCommand`는 단일 `/openclaw` 스타일 명령에 대해 여전히 지원됩니다. `commands.native`를 활성화하면 내장 명령당 하나의 Slack 슬래시 명령을 생성해야 합니다(`/help`와 동일한 이름). Slack의 명령 인수 메뉴는 임시 Block Kit 버튼으로 전달됩니다.
