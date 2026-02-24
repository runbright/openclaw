---
summary: "OpenClaw CLI 참조 - `openclaw` 명령, 하위 명령 및 옵션"
read_when:
  - CLI 명령이나 옵션을 추가하거나 수정할 때
  - 새로운 명령 인터페이스를 문서화할 때
title: "CLI 참조"
---

# CLI 참조

이 페이지는 현재 CLI 동작을 설명합니다. 명령이 변경되면 이 문서를 업데이트하세요.

## 명령 페이지

- [`setup`](/cli/setup)
- [`onboard`](/cli/onboard)
- [`configure`](/cli/configure)
- [`config`](/cli/config)
- [`completion`](/cli/completion)
- [`doctor`](/cli/doctor)
- [`dashboard`](/cli/dashboard)
- [`reset`](/cli/reset)
- [`uninstall`](/cli/uninstall)
- [`update`](/cli/update)
- [`message`](/cli/message)
- [`agent`](/cli/agent)
- [`agents`](/cli/agents)
- [`acp`](/cli/acp)
- [`status`](/cli/status)
- [`health`](/cli/health)
- [`sessions`](/cli/sessions)
- [`gateway`](/cli/gateway)
- [`logs`](/cli/logs)
- [`system`](/cli/system)
- [`models`](/cli/models)
- [`memory`](/cli/memory)
- [`directory`](/cli/directory)
- [`nodes`](/cli/nodes)
- [`devices`](/cli/devices)
- [`node`](/cli/node)
- [`approvals`](/cli/approvals)
- [`sandbox`](/cli/sandbox)
- [`tui`](/cli/tui)
- [`browser`](/cli/browser)
- [`cron`](/cli/cron)
- [`dns`](/cli/dns)
- [`docs`](/cli/docs)
- [`hooks`](/cli/hooks)
- [`webhooks`](/cli/webhooks)
- [`pairing`](/cli/pairing)
- [`qr`](/cli/qr)
- [`plugins`](/cli/plugins) (플러그인 명령)
- [`channels`](/cli/channels)
- [`security`](/cli/security)
- [`skills`](/cli/skills)
- [`daemon`](/cli/daemon) (Gateway 서비스 명령의 레거시 별칭)
- [`clawbot`](/cli/clawbot) (레거시 별칭 네임스페이스)
- [`voicecall`](/cli/voicecall) (플러그인; 설치된 경우)

## 전역 플래그

- `--dev`: `~/.openclaw-dev` 하에 상태를 격리하고 기본 포트를 변경합니다.
- `--profile <name>`: `~/.openclaw-<name>` 하에 상태를 격리합니다.
- `--no-color`: ANSI 색상을 비활성화합니다.
- `--update`: `openclaw update`의 단축키입니다 (소스 설치에서만).
- `-V`, `--version`, `-v`: 버전을 출력하고 종료합니다.

## 출력 스타일링

- ANSI 색상과 진행 표시기는 TTY 세션에서만 렌더링됩니다.
- OSC-8 하이퍼링크는 지원되는 터미널에서 클릭 가능한 링크로 렌더링되며, 그렇지 않으면 일반 URL로 대체됩니다.
- `--json` (지원되는 경우 `--plain`)은 깔끔한 출력을 위해 스타일링을 비활성화합니다.
- `--no-color`는 ANSI 스타일링을 비활성화합니다. `NO_COLOR=1`도 지원됩니다.
- 장시간 실행 명령은 진행 표시기를 보여줍니다 (지원되는 경우 OSC 9;4).

## 색상 팔레트

OpenClaw은 CLI 출력에 랍스터 팔레트를 사용합니다.

- `accent` (#FF5A2D): 제목, 라벨, 주요 강조.
- `accentBright` (#FF7A3D): 명령 이름, 강조.
- `accentDim` (#D14A22): 보조 강조 텍스트.
- `info` (#FF8A5B): 정보 값.
- `success` (#2FBF71): 성공 상태.
- `warn` (#FFB020): 경고, 대체, 주의.
- `error` (#E23D2D): 오류, 실패.
- `muted` (#8B7F77): 강조 해제, 메타데이터.

팔레트 원본: `src/terminal/palette.ts` ("lobster seam"이라고도 함).

## 명령 트리

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

참고: 플러그인은 추가 최상위 명령을 추가할 수 있습니다 (예: `openclaw voicecall`).

## 보안

- `openclaw security audit` - 일반적인 보안 취약점에 대한 설정 + 로컬 상태 감사.
- `openclaw security audit --deep` - 최선의 노력으로 실시간 Gateway 프로브.
- `openclaw security audit --fix` - 안전한 기본값을 강화하고 상태/설정에 chmod 적용.

## 플러그인

확장 프로그램과 설정을 관리합니다:

- `openclaw plugins list` - 플러그인을 검색합니다 (기계 출력에 `--json` 사용).
- `openclaw plugins info <id>` - 플러그인의 세부 정보를 표시합니다.
- `openclaw plugins install <path|.tgz|npm-spec>` - 플러그인을 설치합니다 (또는 `plugins.load.paths`에 플러그인 경로 추가).
- `openclaw plugins enable <id>` / `disable <id>` - `plugins.entries.<id>.enabled`를 토글합니다.
- `openclaw plugins doctor` - 플러그인 로드 오류를 보고합니다.

대부분의 플러그인 변경은 Gateway 재시작이 필요합니다. [/plugin](/tools/plugin)을 참조하세요.

## 메모리

`MEMORY.md` + `memory/*.md`에 대한 벡터 검색:

- `openclaw memory status` - 인덱스 통계를 표시합니다.
- `openclaw memory index` - 메모리 파일을 재인덱싱합니다.
- `openclaw memory search "<query>"` - 메모리에 대한 시맨틱 검색.

## 채팅 슬래시 명령

채팅 메시지는 `/...` 명령(텍스트 및 네이티브)을 지원합니다. [/tools/slash-commands](/tools/slash-commands)를 참조하세요.

주요 기능:

- `/status` 빠른 진단.
- `/config` 영구 설정 변경.
- `/debug` 런타임 전용 설정 재정의 (메모리, 디스크 아님; `commands.debug: true` 필요).

## 설정 + 온보딩

### `setup`

설정 + 워크스페이스를 초기화합니다.

옵션:

- `--workspace <dir>`: 에이전트 워크스페이스 경로 (기본값 `~/.openclaw/workspace`).
- `--wizard`: 온보딩 마법사를 실행합니다.
- `--non-interactive`: 프롬프트 없이 마법사를 실행합니다.
- `--mode <local|remote>`: 마법사 모드.
- `--remote-url <url>`: 원격 Gateway URL.
- `--remote-token <token>`: 원격 Gateway 토큰.

마법사 플래그(`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`)가 있으면 마법사가 자동으로 실행됩니다.

### `onboard`

Gateway, 워크스페이스 및 스킬을 설정하는 대화형 마법사입니다.

옵션:

- `--workspace <dir>`
- `--reset` (마법사 전에 설정 + 자격 증명 + 세션 + 워크스페이스 초기화)
- `--non-interactive`
- `--mode <local|remote>`
- `--flow <quickstart|advanced|manual>` (manual은 advanced의 별칭)
- `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
- `--token-provider <id>` (비대화형; `--auth-choice token`과 함께 사용)
- `--token <token>` (비대화형; `--auth-choice token`과 함께 사용)
- `--token-profile-id <id>` (비대화형; 기본값: `<provider>:manual`)
- `--token-expires-in <duration>` (비대화형; 예: `365d`, `12h`)
- `--anthropic-api-key <key>`
- `--openai-api-key <key>`
- `--mistral-api-key <key>`
- `--openrouter-api-key <key>`
- `--ai-gateway-api-key <key>`
- `--moonshot-api-key <key>`
- `--kimi-code-api-key <key>`
- `--gemini-api-key <key>`
- `--zai-api-key <key>`
- `--minimax-api-key <key>`
- `--opencode-zen-api-key <key>`
- `--custom-base-url <url>` (비대화형; `--auth-choice custom-api-key`와 함께 사용)
- `--custom-model-id <id>` (비대화형; `--auth-choice custom-api-key`와 함께 사용)
- `--custom-api-key <key>` (비대화형; 선택 사항; `--auth-choice custom-api-key`와 함께 사용; 생략 시 `CUSTOM_API_KEY`로 대체)
- `--custom-provider-id <id>` (비대화형; 선택적 커스텀 프로바이더 ID)
- `--custom-compatibility <openai|anthropic>` (비대화형; 선택 사항; 기본값 `openai`)
- `--gateway-port <port>`
- `--gateway-bind <loopback|lan|tailnet|auto|custom>`
- `--gateway-auth <token|password>`
- `--gateway-token <token>`
- `--gateway-password <password>`
- `--remote-url <url>`
- `--remote-token <token>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--install-daemon`
- `--no-install-daemon` (별칭: `--skip-daemon`)
- `--daemon-runtime <node|bun>`
- `--skip-channels`
- `--skip-skills`
- `--skip-health`
- `--skip-ui`
- `--node-manager <npm|pnpm|bun>` (pnpm 권장; bun은 Gateway 런타임에 권장하지 않음)
- `--json`

### `configure`

대화형 구성 마법사 (모델, 채널, 스킬, Gateway).

### `config`

비대화형 설정 도우미 (get/set/unset). `openclaw config`를 하위 명령 없이 실행하면
마법사가 시작됩니다.

하위 명령:

- `config get <path>`: 설정 값을 출력합니다 (점/대괄호 경로).
- `config set <path> <value>`: 값을 설정합니다 (JSON5 또는 원시 문자열).
- `config unset <path>`: 값을 제거합니다.

### `doctor`

상태 점검 + 빠른 수리 (설정 + Gateway + 레거시 서비스).

옵션:

- `--no-workspace-suggestions`: 워크스페이스 메모리 힌트를 비활성화합니다.
- `--yes`: 프롬프트 없이 기본값을 수락합니다 (헤드리스).
- `--non-interactive`: 프롬프트를 건너뜁니다; 안전한 마이그레이션만 적용합니다.
- `--deep`: 추가 Gateway 설치에 대해 시스템 서비스를 스캔합니다.

## 채널 도우미

### `channels`

채팅 채널 계정을 관리합니다 (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (플러그인)/Signal/iMessage/MS Teams).

하위 명령:

- `channels list`: 구성된 채널과 인증 프로필을 표시합니다.
- `channels status`: Gateway 접근성과 채널 상태를 확인합니다 (`--probe`는 추가 검사를 실행합니다; Gateway 상태 프로브는 `openclaw health` 또는 `openclaw status --deep`을 사용합니다).
- 팁: `channels status`는 일반적인 잘못된 구성을 감지할 수 있을 때 수정 제안과 함께 경고를 출력합니다 (그런 다음 `openclaw doctor`를 안내합니다).
- `channels logs`: Gateway 로그 파일에서 최근 채널 로그를 표시합니다.
- `channels add`: 플래그 없이 실행하면 마법사 스타일 설정; 플래그를 사용하면 비대화형 모드로 전환됩니다.
- `channels remove`: 기본적으로 비활성화; `--delete`를 전달하면 프롬프트 없이 설정 항목을 제거합니다.
- `channels login`: 대화형 채널 로그인 (WhatsApp Web만 해당).
- `channels logout`: 채널 세션에서 로그아웃합니다 (지원되는 경우).

공통 옵션:

- `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
- `--account <id>`: 채널 계정 ID (기본값 `default`)
- `--name <label>`: 계정의 표시 이름

`channels login` 옵션:

- `--channel <channel>` (기본값 `whatsapp`; `whatsapp`/`web` 지원)
- `--account <id>`
- `--verbose`

`channels logout` 옵션:

- `--channel <channel>` (기본값 `whatsapp`)
- `--account <id>`

`channels list` 옵션:

- `--no-usage`: 모델 프로바이더 사용량/할당량 스냅샷을 건너뜁니다 (OAuth/API 기반만 해당).
- `--json`: JSON 출력 (`--no-usage`가 설정되지 않은 경우 사용량 포함).

`channels logs` 옵션:

- `--channel <name|all>` (기본값 `all`)
- `--lines <n>` (기본값 `200`)
- `--json`

자세한 내용: [/concepts/oauth](/concepts/oauth)

예시:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### `skills`

사용 가능한 스킬과 준비 상태 정보를 나열하고 검사합니다.

하위 명령:

- `skills list`: 스킬을 나열합니다 (하위 명령이 없을 때 기본값).
- `skills info <name>`: 하나의 스킬에 대한 세부 정보를 표시합니다.
- `skills check`: 준비 완료 vs 누락된 요구사항 요약.

옵션:

- `--eligible`: 준비된 스킬만 표시합니다.
- `--json`: JSON 출력 (스타일링 없음).
- `-v`, `--verbose`: 누락된 요구사항 세부 정보를 포함합니다.

팁: 스킬을 검색, 설치, 동기화하려면 `npx clawhub`를 사용합니다.

### `pairing`

채널 간 DM 페어링 요청을 승인합니다.

하위 명령:

- `pairing list <channel> [--json]`
- `pairing approve <channel> <code> [--notify]`

### `webhooks gmail`

Gmail Pub/Sub 후크 설정 + 실행기. [/automation/gmail-pubsub](/automation/gmail-pubsub)을 참조하세요.

하위 명령:

- `webhooks gmail setup` (`--account <email>` 필수; `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json` 지원)
- `webhooks gmail run` (동일한 플래그에 대한 런타임 재정의)

### `dns setup`

광역 디스커버리 DNS 도우미 (CoreDNS + Tailscale). [/gateway/discovery](/gateway/discovery)를 참조하세요.

옵션:

- `--apply`: CoreDNS 설정을 설치/업데이트합니다 (sudo 필요; macOS만 해당).

## 메시징 + 에이전트

### `message`

통합 아웃바운드 메시징 + 채널 액션.

참조: [/cli/message](/cli/message)

하위 명령:

- `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
- `message thread <create|list|reply>`
- `message emoji <list|upload>`
- `message sticker <send|upload>`
- `message role <info|add|remove>`
- `message channel <info|list>`
- `message member info`
- `message voice status`
- `message event <list|create>`

예시:

- `openclaw message send --target +15555550123 --message "Hi"`
- `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### `agent`

Gateway를 통해 에이전트 턴 하나를 실행합니다 (또는 `--local` 임베디드).

필수:

- `--message <text>`

옵션:

- `--to <dest>` (세션 키 및 선택적 전달용)
- `--session-id <id>`
- `--thinking <off|minimal|low|medium|high|xhigh>` (GPT-5.2 + Codex 모델만 해당)
- `--verbose <on|full|off>`
- `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
- `--local`
- `--deliver`
- `--json`
- `--timeout <seconds>`

### `agents`

격리된 에이전트를 관리합니다 (워크스페이스 + 인증 + 라우팅).

#### `agents list`

구성된 에이전트를 나열합니다.

옵션:

- `--json`
- `--bindings`

#### `agents add [name]`

새로운 격리된 에이전트를 추가합니다. 플래그(또는 `--non-interactive`)가 전달되지 않으면 가이드 마법사를 실행합니다; 비대화형 모드에서는 `--workspace`가 필수입니다.

옵션:

- `--workspace <dir>`
- `--model <id>`
- `--agent-dir <dir>`
- `--bind <channel[:accountId]>` (반복 가능)
- `--non-interactive`
- `--json`

바인딩 스펙은 `channel[:accountId]`를 사용합니다. WhatsApp에서 `accountId`가 생략되면 기본 계정 ID가 사용됩니다.

#### `agents delete <id>`

에이전트를 삭제하고 워크스페이스 + 상태를 정리합니다.

옵션:

- `--force`
- `--json`

### `acp`

IDE를 Gateway에 연결하는 ACP 브릿지를 실행합니다.

전체 옵션 및 예시는 [`acp`](/cli/acp)를 참조하세요.

### `status`

연결된 세션 상태와 최근 수신자를 표시합니다.

옵션:

- `--json`
- `--all` (전체 진단; 읽기 전용, 붙여넣기 가능)
- `--deep` (채널 프로브)
- `--usage` (모델 프로바이더 사용량/할당량 표시)
- `--timeout <ms>`
- `--verbose`
- `--debug` (`--verbose`의 별칭)

참고사항:

- 개요에는 사용 가능한 경우 Gateway + 노드 호스트 서비스 상태가 포함됩니다.

### 사용량 추적

OpenClaw은 OAuth/API 자격 증명이 사용 가능한 경우 프로바이더 사용량/할당량을 표시할 수 있습니다.

표시 위치:

- `/status` (사용 가능한 경우 짧은 프로바이더 사용량 줄 추가)
- `openclaw status --usage` (전체 프로바이더 분석 출력)
- macOS 메뉴 바 (컨텍스트 아래 사용량 섹션)

참고사항:

- 데이터는 프로바이더 사용량 엔드포인트에서 직접 가져옵니다 (추정 아님).
- 프로바이더: Anthropic, GitHub Copilot, OpenAI Codex OAuth, 그리고 해당 프로바이더 플러그인이 활성화된 경우 Gemini CLI/Antigravity.
- 일치하는 자격 증명이 없으면 사용량이 숨겨집니다.
- 세부 사항: [Usage tracking](/concepts/usage-tracking)을 참조하세요.

### `health`

실행 중인 Gateway에서 상태를 가져옵니다.

옵션:

- `--json`
- `--timeout <ms>`
- `--verbose`

### `sessions`

저장된 대화 세션을 나열합니다.

옵션:

- `--json`
- `--verbose`
- `--store <path>`
- `--active <minutes>`

## 초기화 / 제거

### `reset`

로컬 설정/상태를 초기화합니다 (CLI는 설치된 상태로 유지).

옵션:

- `--scope <config|config+creds+sessions|full>`
- `--yes`
- `--non-interactive`
- `--dry-run`

참고사항:

- `--non-interactive`에는 `--scope`와 `--yes`가 필요합니다.

### `uninstall`

Gateway 서비스 + 로컬 데이터를 제거합니다 (CLI는 유지됨).

옵션:

- `--service`
- `--state`
- `--workspace`
- `--app`
- `--all`
- `--yes`
- `--non-interactive`
- `--dry-run`

참고사항:

- `--non-interactive`에는 `--yes`와 명시적 범위(또는 `--all`)가 필요합니다.

## Gateway

### `gateway`

WebSocket Gateway를 실행합니다.

옵션:

- `--port <port>`
- `--bind <loopback|tailnet|lan|auto|custom>`
- `--token <token>`
- `--auth <token|password>`
- `--password <password>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--allow-unconfigured`
- `--dev`
- `--reset` (개발 설정 + 자격 증명 + 세션 + 워크스페이스 초기화)
- `--force` (포트의 기존 리스너 종료)
- `--verbose`
- `--claude-cli-logs`
- `--ws-log <auto|full|compact>`
- `--compact` (`--ws-log compact`의 별칭)
- `--raw-stream`
- `--raw-stream-path <path>`

### `gateway service`

Gateway 서비스를 관리합니다 (launchd/systemd/schtasks).

하위 명령:

- `gateway status` (기본적으로 Gateway RPC를 프로브)
- `gateway install` (서비스 설치)
- `gateway uninstall`
- `gateway start`
- `gateway stop`
- `gateway restart`

참고사항:

- `gateway status`는 기본적으로 서비스의 확인된 포트/설정을 사용하여 Gateway RPC를 프로브합니다 (`--url/--token/--password`로 재정의).
- `gateway status`는 스크립팅을 위한 `--no-probe`, `--deep`, `--json`을 지원합니다.
- `gateway status`는 레거시 또는 추가 Gateway 서비스를 감지할 수 있을 때 표시합니다 (`--deep`은 시스템 수준 스캔 추가). 프로필 이름의 OpenClaw 서비스는 일급으로 취급되며 "추가"로 표시되지 않습니다.
- `gateway status`는 CLI가 사용하는 설정 경로와 서비스가 사용할 가능성이 있는 설정(서비스 환경), 그리고 확인된 프로브 대상 URL을 출력합니다.
- `gateway install|uninstall|start|stop|restart`는 스크립팅을 위한 `--json`을 지원합니다 (기본 출력은 사람이 읽기 쉬운 형태 유지).
- `gateway install`은 기본적으로 Node 런타임을 사용합니다; bun은 **권장하지 않습니다** (WhatsApp/Telegram 버그).
- `gateway install` 옵션: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### `logs`

RPC를 통해 Gateway 파일 로그를 추적합니다.

참고사항:

- TTY 세션은 색상화된 구조화된 뷰를 렌더링합니다; 비 TTY는 일반 텍스트로 대체됩니다.
- `--json`은 줄 구분 JSON을 출력합니다 (한 줄에 로그 이벤트 하나).

예시:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### `gateway <subcommand>`

Gateway CLI 도우미 (RPC 하위 명령에 `--url`, `--token`, `--password`, `--timeout`, `--expect-final` 사용).
`--url`을 전달하면 CLI는 설정이나 환경 자격 증명을 자동 적용하지 않습니다.
`--token` 또는 `--password`를 명시적으로 포함하세요. 명시적 자격 증명이 누락되면 오류가 발생합니다.

하위 명령:

- `gateway call <method> [--params <json>]`
- `gateway health`
- `gateway status`
- `gateway probe`
- `gateway discover`
- `gateway install|uninstall|start|stop|restart`
- `gateway run`

일반 RPC:

- `config.apply` (설정 검증 + 기록 + 재시작 + 깨우기)
- `config.patch` (부분 업데이트 병합 + 재시작 + 깨우기)
- `update.run` (업데이트 실행 + 재시작 + 깨우기)

팁: `config.set`/`config.apply`/`config.patch`를 직접 호출할 때, 설정이 이미 존재하면
`config.get`에서 `baseHash`를 전달하세요.

## 모델

대체 동작과 스캔 전략은 [/concepts/models](/concepts/models)을 참조하세요.

권장 Anthropic 인증 (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

### `models` (루트)

`openclaw models`는 `models status`의 별칭입니다.

루트 옵션:

- `--status-json` (`models status --json`의 별칭)
- `--status-plain` (`models status --plain`의 별칭)

### `models list`

옵션:

- `--all`
- `--local`
- `--provider <name>`
- `--json`
- `--plain`

### `models status`

옵션:

- `--json`
- `--plain`
- `--check` (종료 코드 1=만료/누락, 2=만료 예정)
- `--probe` (구성된 인증 프로필의 실시간 프로브)
- `--probe-provider <name>`
- `--probe-profile <id>` (반복 또는 쉼표 구분)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`

항상 인증 개요와 인증 저장소의 프로필에 대한 OAuth 만료 상태를 포함합니다.
`--probe`는 실시간 요청을 실행합니다 (토큰을 소비하고 속도 제한을 트리거할 수 있음).

### `models set <model>`

`agents.defaults.model.primary`를 설정합니다.

### `models set-image <model>`

`agents.defaults.imageModel.primary`를 설정합니다.

### `models aliases list|add|remove`

옵션:

- `list`: `--json`, `--plain`
- `add <alias> <model>`
- `remove <alias>`

### `models fallbacks list|add|remove|clear`

옵션:

- `list`: `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models image-fallbacks list|add|remove|clear`

옵션:

- `list`: `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models scan`

옵션:

- `--min-params <b>`
- `--max-age-days <days>`
- `--provider <name>`
- `--max-candidates <n>`
- `--timeout <ms>`
- `--concurrency <n>`
- `--no-probe`
- `--yes`
- `--no-input`
- `--set-default`
- `--set-image`
- `--json`

### `models auth add|setup-token|paste-token`

옵션:

- `add`: 대화형 인증 도우미
- `setup-token`: `--provider <name>` (기본값 `anthropic`), `--yes`
- `paste-token`: `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

### `models auth order get|set|clear`

옵션:

- `get`: `--provider <name>`, `--agent <id>`, `--json`
- `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
- `clear`: `--provider <name>`, `--agent <id>`

## 시스템

### `system event`

시스템 이벤트를 큐에 넣고 선택적으로 하트비트를 트리거합니다 (Gateway RPC).

필수:

- `--text <text>`

옵션:

- `--mode <now|next-heartbeat>`
- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system heartbeat last|enable|disable`

하트비트 제어 (Gateway RPC).

옵션:

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system presence`

시스템 프레즌스 항목을 나열합니다 (Gateway RPC).

옵션:

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

## Cron

예약 작업을 관리합니다 (Gateway RPC). [/automation/cron-jobs](/automation/cron-jobs)을 참조하세요.

하위 명령:

- `cron status [--json]`
- `cron list [--all] [--json]` (기본 테이블 출력; 원시 데이터는 `--json` 사용)
- `cron add` (별칭: `create`; `--name`과 `--at` | `--every` | `--cron` 중 정확히 하나, 그리고 `--system-event` | `--message` 중 정확히 하나의 페이로드가 필요)
- `cron edit <id>` (필드 패치)
- `cron rm <id>` (별칭: `remove`, `delete`)
- `cron enable <id>`
- `cron disable <id>`
- `cron runs --id <id> [--limit <n>]`
- `cron run <id> [--force]`

모든 `cron` 명령은 `--url`, `--token`, `--timeout`, `--expect-final`을 지원합니다.

## 노드 호스트

`node`는 **헤드리스 노드 호스트**를 실행하거나 백그라운드 서비스로 관리합니다.
[`openclaw node`](/cli/node)를 참조하세요.

하위 명령:

- `node run --host <gateway-host> --port 18789`
- `node status`
- `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
- `node uninstall`
- `node stop`
- `node restart`

## 노드

`nodes`는 Gateway와 통신하고 페어링된 노드를 대상으로 합니다. [/nodes](/nodes)를 참조하세요.

공통 옵션:

- `--url`, `--token`, `--timeout`, `--json`

하위 명령:

- `nodes status [--connected] [--last-connected <duration>]`
- `nodes describe --node <id|name|ip>`
- `nodes list [--connected] [--last-connected <duration>]`
- `nodes pending`
- `nodes approve <requestId>`
- `nodes reject <requestId>`
- `nodes rename --node <id|name|ip> --name <displayName>`
- `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
- `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (mac 노드 또는 헤드리스 노드 호스트)
- `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (mac만 해당)

카메라:

- `nodes camera list --node <id|name|ip>`
- `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
- `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

캔버스 + 화면:

- `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
- `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
- `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
- `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
- `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

위치:

- `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

## 브라우저

브라우저 제어 CLI (전용 Chrome/Brave/Edge/Chromium). [`openclaw browser`](/cli/browser)와 [Browser tool](/tools/browser)을 참조하세요.

공통 옵션:

- `--url`, `--token`, `--timeout`, `--json`
- `--browser-profile <name>`

관리:

- `browser status`
- `browser start`
- `browser stop`
- `browser reset-profile`
- `browser tabs`
- `browser open <url>`
- `browser focus <targetId>`
- `browser close [targetId]`
- `browser profiles`
- `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
- `browser delete-profile --name <name>`

검사:

- `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
- `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

액션:

- `browser navigate <url> [--target-id <id>]`
- `browser resize <width> <height> [--target-id <id>]`
- `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
- `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
- `browser press <key> [--target-id <id>]`
- `browser hover <ref> [--target-id <id>]`
- `browser drag <startRef> <endRef> [--target-id <id>]`
- `browser select <ref> <values...> [--target-id <id>]`
- `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
- `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
- `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
- `browser console [--level <error|warn|info>] [--target-id <id>]`
- `browser pdf [--target-id <id>]`

## 문서 검색

### `docs [query...]`

실시간 문서 인덱스를 검색합니다.

## TUI

### `tui`

Gateway에 연결된 터미널 UI를 엽니다.

옵션:

- `--url <url>`
- `--token <token>`
- `--password <password>`
- `--session <key>`
- `--deliver`
- `--thinking <level>`
- `--message <text>`
- `--timeout-ms <ms>` (`agents.defaults.timeoutSeconds`가 기본값)
- `--history-limit <n>`
