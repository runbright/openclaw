---
summary: "CLI 온보딩 위자드의 전체 레퍼런스: 모든 단계, 플래그, 설정 필드"
read_when:
  - Looking up a specific wizard step or flag
  - Automating onboarding with non-interactive mode
  - Debugging wizard behavior
title: "온보딩 위자드 레퍼런스"
sidebarTitle: "위자드 레퍼런스"
---

# 온보딩 위자드 레퍼런스

`openclaw onboard` CLI 위자드의 전체 레퍼런스입니다.
상위 수준 개요는 [온보딩 위자드](/start/wizard)를 참조하세요.

## 플로우 세부사항 (로컬 모드)

<Steps>
  <Step title="기존 설정 감지">
    - `~/.openclaw/openclaw.json`이 존재하면 **유지 / 수정 / 리셋** 선택.
    - 위자드를 다시 실행해도 명시적으로 **리셋**을 선택 (또는 `--reset` 전달)하지 않는 한 아무것도 삭제하지 **않습니다**.
    - 설정이 유효하지 않거나 레거시 키가 포함되어 있으면, 위자드가 중지되고
      계속하기 전에 `openclaw doctor`를 실행하라고 요청합니다.
    - 리셋은 `trash`를 사용합니다 (`rm` 사용 안 함) 그리고 범위를 제공합니다:
      - 설정만
      - 설정 + 자격 증명 + 세션
      - 전체 리셋 (워크스페이스도 제거)
  </Step>
  <Step title="모델/인증">
    - **Anthropic API 키 (권장)**: `ANTHROPIC_API_KEY`가 있으면 사용하거나 키를 입력받고 데몬용으로 저장합니다.
    - **Anthropic OAuth (Claude Code CLI)**: macOS에서는 위자드가 키체인 항목 "Claude Code-credentials"를 확인합니다 (launchd 시작이 차단되지 않도록 "항상 허용" 선택); Linux/Windows에서는 `~/.claude/.credentials.json`이 있으면 재사용합니다.
    - **Anthropic 토큰 (setup-token 붙여넣기)**: 브라우저가 있는 기기에서 `claude setup-token`을 실행한 후 토큰을 붙여넣습니다 (이름 지정 가능; 비워두면 기본값).
    - **OpenAI Code (Codex) 구독 (Codex CLI)**: `~/.codex/auth.json`이 존재하면 위자드가 재사용할 수 있습니다.
    - **OpenAI Code (Codex) 구독 (OAuth)**: 브라우저 플로우; `code#state`를 붙여넣습니다.
      - 모델이 미설정이거나 `openai/*`이면 `agents.defaults.model`을 `openai-codex/gpt-5.2`로 설정합니다.
    - **OpenAI API 키**: `OPENAI_API_KEY`가 있으면 사용하거나 키를 입력받고 launchd가 읽을 수 있도록 `~/.openclaw/.env`에 저장합니다.
    - **xAI (Grok) API 키**: `XAI_API_KEY`를 입력받고 xAI를 모델 프로바이더로 설정합니다.
    - **OpenCode Zen (멀티 모델 프록시)**: `OPENCODE_API_KEY` (또는 `OPENCODE_ZEN_API_KEY`, https://opencode.ai/auth 에서 받기)를 입력받습니다.
    - **API 키**: 키를 저장합니다.
    - **Vercel AI Gateway (멀티 모델 프록시)**: `AI_GATEWAY_API_KEY`를 입력받습니다.
    - 자세한 내용: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: Account ID, Gateway ID, `CLOUDFLARE_AI_GATEWAY_API_KEY`를 입력받습니다.
    - 자세한 내용: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: 설정이 자동 작성됩니다.
    - 자세한 내용: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic 호환)**: `SYNTHETIC_API_KEY`를 입력받습니다.
    - 자세한 내용: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: 설정이 자동 작성됩니다.
    - **Kimi Coding**: 설정이 자동 작성됩니다.
    - 자세한 내용: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **건너뛰기**: 아직 인증이 설정되지 않음.
    - 감지된 옵션에서 기본 모델 선택 (또는 provider/model을 수동 입력).
    - 위자드가 모델 확인을 실행하고 설정된 모델이 알 수 없거나 인증이 누락되면 경고합니다.
    - OAuth 자격 증명은 `~/.openclaw/credentials/oauth.json`에; 인증 프로필은 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`에 있습니다 (API 키 + OAuth).
    - 자세한 내용: [/concepts/oauth](/concepts/oauth)
    <Note>
    헤드리스/서버 팁: 브라우저가 있는 기기에서 OAuth를 완료한 후
    `~/.openclaw/credentials/oauth.json` (또는 `$OPENCLAW_STATE_DIR/credentials/oauth.json`)을
    게이트웨이 호스트에 복사하세요.
    </Note>
  </Step>
  <Step title="워크스페이스">
    - 기본값 `~/.openclaw/workspace` (설정 가능).
    - 에이전트 부트스트랩 의식에 필요한 워크스페이스 파일을 시드합니다.
    - 전체 워크스페이스 레이아웃 + 백업 가이드: [에이전트 워크스페이스](/concepts/agent-workspace)
  </Step>
  <Step title="게이트웨이">
    - 포트, 바인드, 인증 모드, tailscale 노출.
    - 인증 권장: 로컬 WS 클라이언트가 인증해야 하므로 루프백에서도 **토큰**을 유지.
    - 모든 로컬 프로세스를 완전히 신뢰하는 경우에만 인증을 비활성화.
    - 비루프백 바인드는 여전히 인증 필요.
  </Step>
  <Step title="채널">
    - [WhatsApp](/channels/whatsapp): 선택적 QR 로그인.
    - [Telegram](/channels/telegram): 봇 토큰.
    - [Discord](/channels/discord): 봇 토큰.
    - [Google Chat](/channels/googlechat): 서비스 계정 JSON + webhook audience.
    - [Mattermost](/channels/mattermost) (플러그인): 봇 토큰 + 기본 URL.
    - [Signal](/channels/signal): 선택적 `signal-cli` 설치 + 계정 설정.
    - [BlueBubbles](/channels/bluebubbles): **iMessage에 권장**; 서버 URL + 비밀번호 + webhook.
    - [iMessage](/channels/imessage): 레거시 `imsg` CLI 경로 + DB 접근.
    - DM 보안: 기본값은 페어링. 첫 DM이 코드를 보냅니다; `openclaw pairing approve <channel> <code>`로 승인하거나 허용 목록을 사용하세요.
  </Step>
  <Step title="데몬 설치">
    - macOS: LaunchAgent
      - 로그인된 사용자 세션 필요; 헤드리스의 경우 커스텀 LaunchDaemon 사용 (제공되지 않음).
    - Linux (그리고 WSL2를 통한 Windows): systemd 사용자 유닛
      - 위자드가 `loginctl enable-linger <user>`로 링거링을 활성화하여 로그아웃 후에도 게이트웨이가 계속 실행되도록 시도합니다.
      - sudo 프롬프트가 나타날 수 있습니다 (`/var/lib/systemd/linger` 쓰기); sudo 없이 먼저 시도합니다.
    - **런타임 선택:** Node (권장; WhatsApp/Telegram에 필요). Bun은 **권장되지 않습니다**.
  </Step>
  <Step title="상태 확인">
    - 게이트웨이를 시작하고 (필요시) `openclaw health`를 실행합니다.
    - 팁: `openclaw status --deep`는 상태 출력에 게이트웨이 상태 프로브를 추가합니다 (도달 가능한 게이트웨이 필요).
  </Step>
  <Step title="스킬 (권장)">
    - 사용 가능한 스킬을 읽고 요구사항을 확인합니다.
    - 노드 매니저 선택: **npm / pnpm** (bun 권장되지 않음).
    - 선택적 의존성을 설치합니다 (일부는 macOS에서 Homebrew 사용).
  </Step>
  <Step title="완료">
    - 요약 + 다음 단계, 추가 기능을 위한 iOS/Android/macOS 앱 포함.
  </Step>
</Steps>

<Note>
GUI가 감지되지 않으면, 위자드는 브라우저를 여는 대신 Control UI용 SSH 포트 포워드 안내를 출력합니다.
Control UI 에셋이 없으면, 위자드가 빌드를 시도합니다; 폴백은 `pnpm ui:build` (UI 의존성 자동 설치)입니다.
</Note>

## 비대화형 모드

온보딩을 자동화하거나 스크립트하려면 `--non-interactive`를 사용:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

머신 판독 가능 요약을 위해 `--json`을 추가.

<Note>
`--json`은 비대화형 모드를 의미하지 **않습니다**. 스크립트에는 `--non-interactive` (그리고 `--workspace`)를 사용하세요.
</Note>

<AccordionGroup>
  <Accordion title="Gemini 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### 에이전트 추가 (비대화형)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## 게이트웨이 위자드 RPC

게이트웨이는 RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`)를 통해 위자드 플로우를 노출합니다.
클라이언트 (macOS 앱, Control UI)는 온보딩 로직을 재구현하지 않고 단계를 렌더링할 수 있습니다.

## Signal 설정 (signal-cli)

위자드는 GitHub 릴리스에서 `signal-cli`를 설치할 수 있습니다:

- 적절한 릴리스 에셋을 다운로드합니다.
- `~/.openclaw/tools/signal-cli/<version>/`에 저장합니다.
- 설정에 `channels.signal.cliPath`를 작성합니다.

참고사항:

- JVM 빌드에는 **Java 21**이 필요합니다.
- 가능하면 네이티브 빌드가 사용됩니다.
- Windows는 WSL2를 사용합니다; signal-cli 설치는 WSL 내 Linux 플로우를 따릅니다.

## 위자드가 작성하는 내용

`~/.openclaw/openclaw.json`의 일반적인 필드:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (Minimax 선택 시)
- `gateway.*` (mode, bind, auth, tailscale)
- `session.dmScope` (동작 세부사항: [CLI 온보딩 레퍼런스](/start/wizard-cli-reference#outputs-and-internals))
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- 채널 허용 목록 (Slack/Discord/Matrix/Microsoft Teams) 프롬프트 중 선택 시 (가능하면 이름을 ID로 해석).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add`는 `agents.list[]` 및 선택적 `bindings`를 작성합니다.

WhatsApp 자격 증명은 `~/.openclaw/credentials/whatsapp/<accountId>/`에 저장됩니다.
세션은 `~/.openclaw/agents/<agentId>/sessions/`에 저장됩니다.

일부 채널은 플러그인으로 제공됩니다. 온보딩 중 선택하면 위자드가
설정하기 전에 설치 (npm 또는 로컬 경로)를 안내합니다.

## 관련 문서

- 위자드 개요: [온보딩 위자드](/start/wizard)
- macOS 앱 온보딩: [온보딩](/start/onboarding)
- 설정 레퍼런스: [게이트웨이 설정](/gateway/configuration)
- 프로바이더: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (레거시)
- 스킬: [스킬](/tools/skills), [스킬 설정](/tools/skills-config)
