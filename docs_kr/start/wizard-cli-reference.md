---
summary: "CLI 온보딩 흐름, 인증/모델 설정, 출력 결과 및 내부 작동 방식에 대한 전체 참조 가이드"
read_when:
  - openclaw onboard 명령의 상세 동작 방식이 궁금할 때
  - 온보딩 결과물을 디버깅하거나 온보딩 클라이언트를 통합할 때
title: "CLI 온보딩 참조 가이드"
sidebarTitle: "CLI 참조 가이드"
---

# CLI 온보딩 참조 가이드

이 페이지는 `openclaw onboard` 명령에 대한 전체 참조 가이드입니다.
요약 가이드는 [온보딩 위저드 (CLI)](/start/wizard)를 참고하세요.

## 위저드가 하는 일

로컬 모드(기본값)는 다음 과정을 안내합니다:

- 모델 및 인증 설정 (OpenAI Code 구독 OAuth, Anthropic API 키 또는 설정 토큰, MiniMax, GLM, Moonshot, AI Gateway 옵션 등)
- 워크스페이스 위치 및 부트스트랩 파일 설정
- 게이트웨이 설정 (포트, 바인딩, 인증, Tailscale)
- 채널 및 프로바이더 설정 (Telegram, WhatsApp, Discord, Google Chat, Mattermost 플러그인, Signal)
- 데몬 설치 (macOS LaunchAgent 또는 Linux/WSL2 systemd 사용자 유닛)
- 상태 점검 (Health check)
- 스킬 설정 (Skills setup)

원격(Remote) 모드는 현재 머신이 다른 곳에서 실행 중인 게이트웨이에 연결하도록 설정합니다. 원격 호스트에 무언가를 설치하거나 수정하지는 않습니다.

## 로컬 흐름 상세 정보

<Steps>
  <Step title="기존 설정 감지">
    - `~/.openclaw/openclaw.json` 파일이 존재하면 유지(Keep), 수정(Modify), 초기화(Reset) 중 선택할 수 있습니다.
    - 위저드를 다시 실행해도 직접 초기화를 선택(또는 `--reset` 전달)하지 않는 한 기존 데이터가 지워지지 않습니다.
    - 설정이 유효하지 않거나 예전 키가 포함되어 있으면 위저드가 중단됩니다. 이 경우 `openclaw doctor`를 먼저 실행해야 합니다.
    - 초기화 시 `trash` 명령을 사용하며 다음 범위를 선택할 수 있습니다:
      - 설정 파일만 (Config only)
      - 설정 + 인증 정보 + 세션 (Config + credentials + sessions)
      - 전체 초기화 (Full reset, 워크스페이스까지 포함)
  </Step>
  <Step title="모델 및 인증">
    - 상세 대조표는 [인증 및 모델 옵션](#인증-및-모델-옵션)을 참고하세요.
  </Step>
  <Step title="워크스페이스">
    - 기본값은 `~/.openclaw/workspace`입니다 (변경 가능).
    - 첫 실행 부트스트래핑 리추얼에 필요한 파일들을 생성합니다.
    - 워크스페이스 구조: [에이전트 워크스페이스](/concepts/agent-workspace)를 참고하세요.
  </Step>
  <Step title="게이트웨이">
    - 포트, 바인딩, 인증 모드, Tailscale 노출 여부를 묻습니다.
    - 권장 사항: 루프백 접속이더라도 토큰 인증을 활성화하여 로컬 클라이언트들이 인증을 거치도록 하세요.
    - 인증을 비활성화하는 것은 해당 머신의 모든 프로세스를 완전히 신뢰할 수 있을 때만 수행하세요.
    - 루프백이 아닌 바인딩은 항상 인증이 필요합니다.
  </Step>
  <Step title="채널">
    - [WhatsApp](/channels/whatsapp): 선택적 QR 로그인
    - [Telegram](/channels/telegram): 봇 토큰
    - [Discord](/channels/discord): 봇 토큰
    - [Google Chat](/channels/googlechat): 서비스 계정 JSON + 웹훅 오디언스
    - [Mattermost](/channels/mattermost) 플러그인: 봇 토큰 + 베이스 URL
    - [Signal](/channels/signal): 선택적 `signal-cli` 설치 및 계정 설정
    - [BlueBubbles](/channels/bluebubbles): iMessage용 권장 방식. 서버 URL + 비밀번호 + 웹훅
    - [iMessage](/channels/imessage): 레거시 `imsg` CLI 경로 + DB 액세스
    - DM 보안: 기본은 페어링(pairing) 방식입니다. 첫 DM 시 코드가 전송되며, `openclaw pairing approve <채널> <코드>` 명령으로 승인하거나 허용 목록을 사용해야 합니다.
  </Step>
  <Step title="데몬 설치">
    - macOS: LaunchAgent
      - 로그인된 사용자 세션이 필요합니다. 헤드리스 환경을 위해서는 커스텀 LaunchDaemon(프로젝트에서 제공 안 함)이 필요합니다.
    - Linux 및 Windows(WSL2): systemd 사용자 유닛
      - 로그아웃 후에도 게이트웨이가 유지되도록 위저드에서 `loginctl enable-linger <사용자>` 활성화를 시도합니다.
      - sudo 권한이 필요할 수 있으며(` /var/lib/systemd/linger` 쓰기 권한), 먼저 권한 없이 시도합니다.
    - 런타임 선택: Node(권장; WhatsApp 및 Telegram 사용 시 필수). Bun은 권장하지 않습니다.
  </Step>
  <Step title="상태 점검">
    - 필요한 경우 게이트웨이를 시작하고 `openclaw health`를 실행합니다.
    - `openclaw status --deep` 명령은 상태 출력 결과에 게이트웨이 헬스 체크 결과를 추가합니다.
  </Step>
  <Step title="스킬">
    - 사용 가능한 스킬을 읽고 요구 사항을 확인합니다.
    - 패키지 매니저(npm 또는 pnpm)를 선택하도록 합니다 (Bun 비권장).
    - 선택적 의존성 파일들을 설치합니다 (macOS에서는 일부 Homebrew 사용).
  </Step>
  <Step title="완료">
    - 요약 정보와 함께 iOS, Android, macOS 앱 등의 다음 단계를 안내합니다.
  </Step>
</Steps>

<Note>
GUI 환경이 감지되지 않으면 위저드는 브라우저를 여는 대신 제어 UI 접속을 위한 SSH 포트 포워딩 안내를 출력합니다.
제어 UI 에셋이 없으면 위저드가 직접 빌드를 시도하며, 안 되면 `pnpm ui:build`를 실행합니다 (UI 의존성 자동 설치).
</Note>

## 원격(Remote) 모드 상세 정보

원격 모드는 현재 머신이 다른 곳에서 실행 중인 게이트웨이에 연결하도록 설정합니다.

<Info>
원격 모드는 원격 호스트에 어떠한 것도 설치하거나 수정하지 않습니다.
</Info>

설정 항목:

- 원격 게이트웨이 URL (`ws://...`)
- 토큰 (원격 게이트웨이 인증이 필요한 경우 권장)

<Note>
- 게이트웨이가 루프백 전용인 경우 SSH 터널링이나 Tailnet을 사용하세요.
- 감지 힌트:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## 인증 및 모델 옵션

<AccordionGroup>
  <Accordion title="Anthropic API 키 (권장)">
    `ANTHROPIC_API_KEY`가 있으면 사용하고, 없으면 키를 입력받아 데몬에서 사용할 수 있도록 저장합니다.
  </Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: Keychain 앱의 "Claude Code-credentials" 항목을 확인합니다.
    - Linux 및 Windows: `~/.claude/.credentials.json` 파일이 있으면 재사용합니다.

    macOS에서는 launchd 실행 시 차단되지 않도록 "항상 허용(Always Allow)"을 선택하세요.
  </Accordion>
  <Accordion title="Anthropic 토큰 (setup-token 붙여넣기)">
    아무 머신에서나 `claude setup-token`을 실행한 뒤 해당 토큰을 붙여넣으세요. 이름을 지정할 수 있으며 비워두면 기본값이 사용됩니다.
  </Accordion>
  <Accordion title="OpenAI Code 구독 (Codex CLI 재사용)">
    `~/.codex/auth.json`이 존재하면 재사용할 수 있습니다.
  </Accordion>
  <Accordion title="OpenAI Code 구독 (OAuth)">
    브라우저 인증 흐름입니다. `code#state` 값을 붙여넣으세요. 모델이 설정되지 않았거나 `openai/*`인 경우 기본 모델을 `openai-codex/gpt-5.3-codex`로 설정합니다.
  </Accordion>
  <Accordion title="OpenAI API 키">
    `OPENAI_API_KEY`가 있으면 사용하고, 없으면 키를 입력받아 launchd가 읽을 수 있도록 `~/.openclaw/.env`에 저장합니다. 모델이 설정되지 않은 경우 기본값을 `openai/gpt-5.1-codex` 등으로 설정합니다.
  </Accordion>
  <Accordion title="xAI (Grok) API 키">
    `XAI_API_KEY` 입력을 요청하고 xAI를 모델 프로바이더로 설정합니다.
  </Accordion>
  <Accordion title="OpenCode Zen">
    `OPENCODE_API_KEY` (또는 `OPENCODE_ZEN_API_KEY`) 입력을 요청합니다. 설정 URL: [opencode.ai/auth](https://opencode.ai/auth).
  </Accordion>
  <Accordion title="API 키 (일반)">
    키를 직접 입력받아 저장합니다.
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    `AI_GATEWAY_API_KEY` 입력을 요청합니다. 상세 내용: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    계정 ID, 게이트웨이 ID 및 `CLOUDFLARE_AI_GATEWAY_API_KEY` 입력을 요청합니다. 상세 내용: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  </Accordion>
  <Accordion title="MiniMax M2.1">
    설정 파일이 자동 생성됩니다. 상세 내용: [MiniMax](/providers/minimax).
  </Accordion>
  <Accordion title="Synthetic (Anthropic 호환)">
    `SYNTHETIC_API_KEY` 입력을 요청합니다. 상세 내용: [Synthetic](/providers/synthetic).
  </Accordion>
  <Accordion title="Moonshot 및 Kimi Coding">
    Moonshot (Kimi K2) 및 Kimi Coding 설정이 자동 생성됩니다. 상세 내용: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  </Accordion>
  <Accordion title="커스텀 프로바이더 (Custom provider)">
    OpenAI 호환 및 Anthropic 호환 엔드포인트가 지원됩니다. 자동화를 위한 플래그들(`--custom-base-url`, `--custom-model-id` 등)을 지원합니다.
  </Accordion>
  <Accordion title="건너뛰기">
    인증 설정을 구성하지 않은 채로 둡니다.
  </Accordion>
</AccordionGroup>

모델 동작:

- 감지된 옵션 중 기본 모델을 선택하거나 프로바이더와 모델명을 수동 입력합니다.
- 위저드는 모델 체크를 실행하여 설정된 모델이 알 수 없거나 인증 정보가 누락된 경우 경고를 표시합니다.

인증 및 프로필 경로:

- OAuth 인증 정보: `~/.openclaw/credentials/oauth.json`
- 인증 프로필 (API 키 + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
GUI가 없는 서버 팁: 브라우저가 있는 머신에서 OAuth를 완료한 뒤, 생성된 `oauth.json` 파일을 게이트웨이 호스트의 동일한 경로로 복사하세요.
</Note>

## 출력 결과 및 내부 작동 방식

`~/.openclaw/openclaw.json` 파일의 주요 필드:

- `agents.defaults.workspace` (워크스페이스 경로)
- `agents.defaults.model` (기본 모델)
- `gateway.*` (모드, 바인딩, 인증, Tailscale 설정)
- `session.dmScope` (로컬 온보딩 시 기본값은 `per-channel-peer`로 설정됨)
- `channels.*` (채널별 토큰 및 설정)
- 채널 허용 목록 (Slack, Discord, Matrix, Microsoft Teams 등)
- `skills.install.nodeManager` (패키지 매니저)
- `wizard.*` (마지막 실행 기록: 시간, 버전, 명령어 등)

`openclaw agents add` 명령은 `agents.list[]` 항목과 선택적인 `bindings` 정보를 기록합니다.
WhatsApp 인증 정보는 `~/.openclaw/credentials/whatsapp/<accountId>/` 아래에, 세션 파일은 `~/.openclaw/agents/<agentId>/sessions/` 아래에 저장됩니다.

<Note>
일부 채널은 플러그인 형태로 제공됩니다. 온보딩 중 선택하면 채널 설정을 시작하기 전에 플러그인을 먼저 설치(npm 또는 로컬 경로)하도록 안내합니다.
</Note>

게이트웨이 위저드 RPC:
제어 UI나 macOS 앱 같은 클라이언트들이 자체적으로 온보딩 로직을 구현하지 않고도 이 인터페이스를 통해 단계를 렌더링할 수 있습니다.

Signal 설정 동작:
- 적합한 릴리스 에셋을 다운로드하여 `~/.openclaw/tools/signal-cli/` 아래에 저장하고 설정을 업데이트합니다.
- JVM 빌드는 Java 21이 필요하며, 사용 가능한 경우 네이티브 빌드를 사용합니다. Windows는 WSL2를 통해 Linux용 Signal-cli 흐름을 따릅니다.

## 관련 문서

- 온보딩 허브: [온보딩 위저드 (CLI)](/start/wizard)
- 자동화 및 스크립트: [CLI 자동화](/start/wizard-cli-automation)
- 명령 참조: [`openclaw onboard`](/cli/onboard)
