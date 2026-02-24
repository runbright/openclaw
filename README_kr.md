# OpenClaw — 개인 AI 어시스턴트

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png">
        <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500">
    </picture>
</p>

<p align="center">
  <strong>EXFOLIATE! EXFOLIATE!</strong>
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw/actions/workflows/ci.yml?branch=main"><img src="https://img.shields.io/github/actions/workflow/status/openclaw/openclaw/ci.yml?branch=main&style=for-the-badge" alt="CI status"></a>
  <a href="https://github.com/openclaw/openclaw/releases"><img src="https://img.shields.io/github/v/release/openclaw/openclaw?include_prereleases&style=for-the-badge" alt="GitHub release"></a>
  <a href="https://discord.gg/clawd"><img src="https://img.shields.io/discord/1456350064065904867?label=Discord&logo=discord&logoColor=white&color=5865F2&style=for-the-badge" alt="Discord"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
</p>

**OpenClaw**는 자신의 기기에서 실행하는 _개인 AI 어시스턴트_입니다.
이미 사용하고 있는 채널(WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, WebChat)은 물론 BlueBubbles, Matrix, Zalo, Zalo Personal 같은 확장 채널에서도 응답합니다. macOS/iOS/Android에서 음성으로 말하고 들을 수 있으며, 제어할 수 있는 라이브 Canvas를 렌더링할 수 있습니다. 게이트웨이는 단지 컨트롤 플레인일 뿐 - 제품은 어시스턴트입니다.

로컬에서 빠르고 항상 켜져 있는 개인 단일 사용자 어시스턴트를 원한다면, 바로 이것입니다.

[웹사이트](https://openclaw.ai) · [문서](https://docs.openclaw.ai) · [비전](VISION.md) · [DeepWiki](https://deepwiki.com/openclaw/openclaw) · [시작하기](https://docs.openclaw.ai/start/getting-started) · [업데이트](https://docs.openclaw.ai/install/updating) · [쇼케이스](https://docs.openclaw.ai/start/showcase) · [FAQ](https://docs.openclaw.ai/start/faq) · [마법사](https://docs.openclaw.ai/start/wizard) · [Nix](https://github.com/openclaw/nix-openclaw) · [Docker](https://docs.openclaw.ai/install/docker) · [Discord](https://discord.gg/clawd)

권장 설정: 터미널에서 온보딩 마법사(`openclaw onboard`)를 실행하세요.
마법사가 게이트웨이, 워크스페이스, 채널, 스킬 설정을 단계별로 안내합니다. CLI 마법사가 권장 경로이며 **macOS, Linux, Windows(WSL2 경유; 강력 권장)**에서 작동합니다.
npm, pnpm, bun과 함께 사용 가능합니다.
새로 설치하시나요? 여기서 시작하세요: [시작하기](https://docs.openclaw.ai/start/getting-started)

## 스폰서

| OpenAI                                                            | Blacksmith                                                                   |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| [![OpenAI](docs/assets/sponsors/openai.svg)](https://openai.com/) | [![Blacksmith](docs/assets/sponsors/blacksmith.svg)](https://blacksmith.sh/) |

**구독 (OAuth):**

- **[OpenAI](https://openai.com/)** (ChatGPT/Codex)

모델 참고: 어떤 모델이든 지원되지만, 긴 컨텍스트 강점과 더 나은 프롬프트 주입 저항을 위해 **Anthropic Pro/Max (100/200) + Opus 4.6**을 강력히 권장합니다. [온보딩](https://docs.openclaw.ai/start/onboarding)을 참조하세요.

## 모델 (선택 + 인증)

- 모델 설정 + CLI: [모델](https://docs.openclaw.ai/concepts/models)
- 인증 프로필 로테이션 (OAuth vs API 키) + 폴백: [모델 페일오버](https://docs.openclaw.ai/concepts/model-failover)

## 설치 (권장)

런타임: **Node 22 이상**.

```bash
npm install -g openclaw@latest
# 또는: pnpm add -g openclaw@latest

openclaw onboard --install-daemon
```

마법사가 게이트웨이 데몬(launchd/systemd 사용자 서비스)을 설치하여 계속 실행되도록 합니다.

## 빠른 시작 (요약)

런타임: **Node 22 이상**.

전체 초보자 가이드 (인증, 페어링, 채널): [시작하기](https://docs.openclaw.ai/start/getting-started)

```bash
openclaw onboard --install-daemon

openclaw gateway --port 18789 --verbose

# 메시지 보내기
openclaw message send --to +1234567890 --message "Hello from OpenClaw"

# 어시스턴트와 대화 (선택적으로 연결된 채널로 전달: WhatsApp/Telegram/Slack/Discord/Google Chat/Signal/iMessage/BlueBubbles/Microsoft Teams/Matrix/Zalo/Zalo Personal/WebChat)
openclaw agent --message "Ship checklist" --thinking high
```

업그레이드하시나요? [업데이트 가이드](https://docs.openclaw.ai/install/updating) (그리고 `openclaw doctor`를 실행하세요).

## 개발 채널

- **stable**: 태그된 릴리스만 (`vYYYY.M.D` 또는 `vYYYY.M.D-<patch>`), npm dist-tag `latest`.
- **beta**: 프리릴리스 태그 (`vYYYY.M.D-beta.N`), npm dist-tag `beta` (macOS 앱이 없을 수 있음).
- **dev**: `main`의 이동 헤드, npm dist-tag `dev` (게시된 경우).

채널 전환 (git + npm): `openclaw update --channel stable|beta|dev`.
상세: [개발 채널](https://docs.openclaw.ai/install/development-channels).

## 소스에서 빌드 (개발)

소스에서 빌드할 때는 `pnpm`을 선호합니다. Bun은 TypeScript를 직접 실행하기 위한 선택 사항입니다.

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw

pnpm install
pnpm ui:build # 첫 실행 시 UI 의존성 자동 설치
pnpm build

pnpm openclaw onboard --install-daemon

# 개발 루프 (TS 변경 시 자동 새로고침)
pnpm gateway:watch
```

참고: `pnpm openclaw ...`는 TypeScript를 직접 실행합니다 (`tsx` 경유). `pnpm build`는 Node / 패키지된 `openclaw` 바이너리로 실행하기 위한 `dist/`를 생성합니다.

## 보안 기본값 (DM 접근)

OpenClaw는 실제 메시징 표면에 연결됩니다. 인바운드 DM을 **신뢰하지 않는 입력**으로 취급하세요.

전체 보안 가이드: [보안](https://docs.openclaw.ai/gateway/security)

Telegram/WhatsApp/Signal/iMessage/Microsoft Teams/Discord/Google Chat/Slack의 기본 동작:

- **DM 페어링** (`dmPolicy="pairing"` / `channels.discord.dmPolicy="pairing"` / `channels.slack.dmPolicy="pairing"`; 레거시: `channels.discord.dm.policy`, `channels.slack.dm.policy`): 알 수 없는 발신자는 짧은 페어링 코드를 받으며 봇은 메시지를 처리하지 않습니다.
- 승인: `openclaw pairing approve <channel> <code>` (그러면 발신자가 로컬 허용 목록 스토어에 추가됩니다).
- 공개 인바운드 DM은 명시적 옵트인이 필요합니다: `dmPolicy="open"`으로 설정하고 채널 허용 목록에 `"*"`를 포함하세요 (`allowFrom` / `channels.discord.allowFrom` / `channels.slack.allowFrom`; 레거시: `channels.discord.dm.allowFrom`, `channels.slack.dm.allowFrom`).

`openclaw doctor`를 실행하여 위험한/잘못된 DM 정책을 확인하세요.

## 주요 기능

- **[로컬 우선 게이트웨이](https://docs.openclaw.ai/gateway)** — 세션, 채널, 도구, 이벤트를 위한 단일 컨트롤 플레인.
- **[멀티채널 인박스](https://docs.openclaw.ai/channels)** — WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, BlueBubbles (iMessage), iMessage (레거시), Microsoft Teams, Matrix, Zalo, Zalo Personal, WebChat, macOS, iOS/Android.
- **[멀티에이전트 라우팅](https://docs.openclaw.ai/gateway/configuration)** — 인바운드 채널/계정/피어를 격리된 에이전트(워크스페이스 + 에이전트별 세션)로 라우팅.
- **[Voice Wake](https://docs.openclaw.ai/nodes/voicewake) + [Talk Mode](https://docs.openclaw.ai/nodes/talk)** — ElevenLabs와 함께 macOS/iOS/Android에서 상시 음성.
- **[라이브 Canvas](https://docs.openclaw.ai/platforms/mac/canvas)** — [A2UI](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui)를 사용한 에이전트 구동 시각적 워크스페이스.
- **[퍼스트클래스 도구](https://docs.openclaw.ai/tools)** — 브라우저, canvas, 노드, cron, 세션, Discord/Slack 액션.
- **[컴패니언 앱](https://docs.openclaw.ai/platforms/macos)** — macOS 메뉴 바 앱 + iOS/Android [노드](https://docs.openclaw.ai/nodes).
- **[온보딩](https://docs.openclaw.ai/start/wizard) + [스킬](https://docs.openclaw.ai/tools/skills)** — 번들/관리/워크스페이스 스킬을 갖춘 마법사 기반 설정.

## Star 히스토리

[![Star History Chart](https://api.star-history.com/svg?repos=openclaw/openclaw&type=date&legend=top-left)](https://www.star-history.com/#openclaw/openclaw&type=date&legend=top-left)

## 지금까지 만든 모든 것

### 핵심 플랫폼

- [게이트웨이 WS 컨트롤 플레인](https://docs.openclaw.ai/gateway): 세션, 프레즌스, 설정, cron, 웹훅, [Control UI](https://docs.openclaw.ai/web), [Canvas 호스트](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui).
- [CLI 표면](https://docs.openclaw.ai/tools/agent-send): gateway, agent, send, [마법사](https://docs.openclaw.ai/start/wizard), [doctor](https://docs.openclaw.ai/gateway/doctor).
- [Pi 에이전트 런타임](https://docs.openclaw.ai/concepts/agent): 도구 스트리밍 및 블록 스트리밍이 있는 RPC 모드.
- [세션 모델](https://docs.openclaw.ai/concepts/session): 직접 채팅을 위한 `main`, 그룹 격리, 활성화 모드, 큐 모드, 회신. 그룹 규칙: [그룹](https://docs.openclaw.ai/concepts/groups).
- [미디어 파이프라인](https://docs.openclaw.ai/nodes/images): 이미지/오디오/비디오, 트랜스크립션 훅, 크기 제한, 임시 파일 수명주기. 오디오 상세: [오디오](https://docs.openclaw.ai/nodes/audio).

### 채널

- [채널](https://docs.openclaw.ai/channels): [WhatsApp](https://docs.openclaw.ai/channels/whatsapp) (Baileys), [Telegram](https://docs.openclaw.ai/channels/telegram) (grammY), [Slack](https://docs.openclaw.ai/channels/slack) (Bolt), [Discord](https://docs.openclaw.ai/channels/discord) (discord.js), [Google Chat](https://docs.openclaw.ai/channels/googlechat) (Chat API), [Signal](https://docs.openclaw.ai/channels/signal) (signal-cli), [BlueBubbles](https://docs.openclaw.ai/channels/bluebubbles) (iMessage, 권장), [iMessage](https://docs.openclaw.ai/channels/imessage) (레거시 imsg), [Microsoft Teams](https://docs.openclaw.ai/channels/msteams) (확장), [Matrix](https://docs.openclaw.ai/channels/matrix) (확장), [Zalo](https://docs.openclaw.ai/channels/zalo) (확장), [Zalo Personal](https://docs.openclaw.ai/channels/zalouser) (확장), [WebChat](https://docs.openclaw.ai/web/webchat).
- [그룹 라우팅](https://docs.openclaw.ai/concepts/group-messages): 멘션 게이팅, 답장 태그, 채널별 청킹 및 라우팅. 채널 규칙: [채널](https://docs.openclaw.ai/channels).

### 앱 + 노드

- [macOS 앱](https://docs.openclaw.ai/platforms/macos): 메뉴 바 컨트롤 플레인, [Voice Wake](https://docs.openclaw.ai/nodes/voicewake)/PTT, [Talk Mode](https://docs.openclaw.ai/nodes/talk) 오버레이, [WebChat](https://docs.openclaw.ai/web/webchat), 디버그 도구, [원격 게이트웨이](https://docs.openclaw.ai/gateway/remote) 제어.
- [iOS 노드](https://docs.openclaw.ai/platforms/ios): [Canvas](https://docs.openclaw.ai/platforms/mac/canvas), [Voice Wake](https://docs.openclaw.ai/nodes/voicewake), [Talk Mode](https://docs.openclaw.ai/nodes/talk), 카메라, 화면 녹화, Bonjour 페어링.
- [Android 노드](https://docs.openclaw.ai/platforms/android): [Canvas](https://docs.openclaw.ai/platforms/mac/canvas), [Talk Mode](https://docs.openclaw.ai/nodes/talk), 카메라, 화면 녹화, 선택적 SMS.
- [macOS 노드 모드](https://docs.openclaw.ai/nodes): system.run/notify + canvas/카메라 노출.

### 도구 + 자동화

- [브라우저 제어](https://docs.openclaw.ai/tools/browser): 전용 openclaw Chrome/Chromium, 스냅샷, 액션, 업로드, 프로필.
- [Canvas](https://docs.openclaw.ai/platforms/mac/canvas): [A2UI](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui) push/reset, eval, snapshot.
- [노드](https://docs.openclaw.ai/nodes): 카메라 스냅/클립, 화면 녹화, [location.get](https://docs.openclaw.ai/nodes/location-command), 알림.
- [Cron + 깨우기](https://docs.openclaw.ai/automation/cron-jobs); [웹훅](https://docs.openclaw.ai/automation/webhook); [Gmail Pub/Sub](https://docs.openclaw.ai/automation/gmail-pubsub).
- [스킬 플랫폼](https://docs.openclaw.ai/tools/skills): 번들, 관리, 워크스페이스 스킬 + 설치 게이팅 + UI.

### 런타임 + 안전

- [채널 라우팅](https://docs.openclaw.ai/concepts/channel-routing), [재시도 정책](https://docs.openclaw.ai/concepts/retry), [스트리밍/청킹](https://docs.openclaw.ai/concepts/streaming).
- [프레즌스](https://docs.openclaw.ai/concepts/presence), [타이핑 표시](https://docs.openclaw.ai/concepts/typing-indicators), [사용량 추적](https://docs.openclaw.ai/concepts/usage-tracking).
- [모델](https://docs.openclaw.ai/concepts/models), [모델 페일오버](https://docs.openclaw.ai/concepts/model-failover), [세션 프루닝](https://docs.openclaw.ai/concepts/session-pruning).
- [보안](https://docs.openclaw.ai/gateway/security) 및 [문제 해결](https://docs.openclaw.ai/channels/troubleshooting).

### 운영 + 패키징

- [Control UI](https://docs.openclaw.ai/web) + [WebChat](https://docs.openclaw.ai/web/webchat)는 게이트웨이에서 직접 서비스됩니다.
- [Tailscale Serve/Funnel](https://docs.openclaw.ai/gateway/tailscale) 또는 토큰/비밀번호 인증이 있는 [SSH 터널](https://docs.openclaw.ai/gateway/remote).
- [Nix 모드](https://docs.openclaw.ai/install/nix): 선언적 설정; [Docker](https://docs.openclaw.ai/install/docker) 기반 설치.
- [Doctor](https://docs.openclaw.ai/gateway/doctor) 마이그레이션, [로깅](https://docs.openclaw.ai/logging).

## 작동 방식 (요약)

```
WhatsApp / Telegram / Slack / Discord / Google Chat / Signal / iMessage / BlueBubbles / Microsoft Teams / Matrix / Zalo / Zalo Personal / WebChat
               |
               v
+-------------------------------+
|            Gateway            |
|       (control plane)         |
|     ws://127.0.0.1:18789      |
+---------------+---------------+
               |
               +-- Pi agent (RPC)
               +-- CLI (openclaw ...)
               +-- WebChat UI
               +-- macOS app
               +-- iOS / Android nodes
```

## 주요 서브시스템

- **[게이트웨이 WebSocket 네트워크](https://docs.openclaw.ai/concepts/architecture)** — 클라이언트, 도구, 이벤트를 위한 단일 WS 컨트롤 플레인 (운영: [게이트웨이 런북](https://docs.openclaw.ai/gateway)).
- **[Tailscale 노출](https://docs.openclaw.ai/gateway/tailscale)** — 게이트웨이 대시보드 + WS를 위한 Serve/Funnel (원격 접근: [원격](https://docs.openclaw.ai/gateway/remote)).
- **[브라우저 제어](https://docs.openclaw.ai/tools/browser)** — CDP 제어가 있는 openclaw 관리 Chrome/Chromium.
- **[Canvas + A2UI](https://docs.openclaw.ai/platforms/mac/canvas)** — 에이전트 구동 시각적 워크스페이스 (A2UI 호스트: [Canvas/A2UI](https://docs.openclaw.ai/platforms/mac/canvas#canvas-a2ui)).
- **[Voice Wake](https://docs.openclaw.ai/nodes/voicewake) + [Talk Mode](https://docs.openclaw.ai/nodes/talk)** — 상시 음성 및 연속 대화.
- **[노드](https://docs.openclaw.ai/nodes)** — Canvas, 카메라 스냅/클립, 화면 녹화, `location.get`, 알림, macOS 전용 `system.run`/`system.notify`.

## Tailscale 접근 (게이트웨이 대시보드)

OpenClaw는 게이트웨이가 루프백에 바인드된 상태로 Tailscale **Serve** (tailnet 전용) 또는 **Funnel** (공개)을 자동 설정할 수 있습니다. `gateway.tailscale.mode` 설정:

- `off`: Tailscale 자동화 없음 (기본값).
- `serve`: `tailscale serve`를 통한 tailnet 전용 HTTPS (기본적으로 Tailscale 아이덴티티 헤더 사용).
- `funnel`: `tailscale funnel`을 통한 공개 HTTPS (공유 비밀번호 인증 필요).

참고:

- Serve/Funnel이 활성화되면 `gateway.bind`는 `loopback`으로 유지해야 합니다 (OpenClaw가 이를 강제합니다).
- Serve는 `gateway.auth.mode: "password"` 또는 `gateway.auth.allowTailscale: false`를 설정하여 비밀번호를 요구하도록 강제할 수 있습니다.
- Funnel은 `gateway.auth.mode: "password"`가 설정되지 않으면 시작을 거부합니다.
- 선택 사항: 종료 시 Serve/Funnel을 취소하는 `gateway.tailscale.resetOnExit`.

상세: [Tailscale 가이드](https://docs.openclaw.ai/gateway/tailscale) · [웹 표면](https://docs.openclaw.ai/web)

## 원격 게이트웨이 (Linux 추천)

게이트웨이를 소규모 Linux 인스턴스에서 실행하는 것은 완전히 괜찮습니다. 클라이언트(macOS 앱, CLI, WebChat)는 **Tailscale Serve/Funnel** 또는 **SSH 터널**을 통해 연결할 수 있으며, 필요할 때 디바이스 노드(macOS/iOS/Android)를 페어링하여 디바이스 로컬 작업을 실행할 수 있습니다.

- **게이트웨이 호스트**는 기본적으로 exec 도구와 채널 연결을 실행합니다.
- **디바이스 노드**는 `node.invoke`를 통해 디바이스 로컬 작업(`system.run`, 카메라, 화면 녹화, 알림)을 실행합니다.
  요약: exec은 게이트웨이가 있는 곳에서 실행되고; 디바이스 작업은 디바이스가 있는 곳에서 실행됩니다.

상세: [원격 접근](https://docs.openclaw.ai/gateway/remote) · [노드](https://docs.openclaw.ai/nodes) · [보안](https://docs.openclaw.ai/gateway/security)

## 게이트웨이 프로토콜을 통한 macOS 권한

macOS 앱은 **노드 모드**로 실행할 수 있으며 게이트웨이 WebSocket을 통해 기능 + 권한 맵을 광고합니다 (`node.list` / `node.describe`). 그러면 클라이언트가 `node.invoke`를 통해 로컬 작업을 실행할 수 있습니다:

- `system.run`은 로컬 명령을 실행하고 stdout/stderr/exit code를 반환합니다; 화면 녹화 권한이 필요한 경우 `needsScreenRecording: true`를 설정하세요 (그렇지 않으면 `PERMISSION_MISSING`이 발생합니다).
- `system.notify`는 사용자 알림을 게시하며 알림이 거부되면 실패합니다.
- `canvas.*`, `camera.*`, `screen.record`, `location.get`도 `node.invoke`를 통해 라우팅되며 TCC 권한 상태를 따릅니다.

상위 bash (호스트 권한)는 macOS TCC와 별개입니다:

- `/elevated on|off`를 사용하여 활성화 + 허용 목록에 있을 때 세션별 상위 접근을 토글합니다.
- 게이트웨이는 `sessions.patch` (WS 메서드)를 통해 `thinkingLevel`, `verboseLevel`, `model`, `sendPolicy`, `groupActivation`과 함께 세션별 토글을 유지합니다.

상세: [노드](https://docs.openclaw.ai/nodes) · [macOS 앱](https://docs.openclaw.ai/platforms/macos) · [게이트웨이 프로토콜](https://docs.openclaw.ai/concepts/architecture)

## 에이전트 간 통신 (sessions\_\* 도구)

- 채팅 표면 간 전환 없이 세션 간 작업을 조정하는 데 사용합니다.
- `sessions_list` — 활성 세션(에이전트)과 메타데이터를 검색합니다.
- `sessions_history` — 세션의 트랜스크립트 로그를 가져옵니다.
- `sessions_send` — 다른 세션에 메시지를 보냅니다; 선택적 회신 핑퐁 + 공지 단계 (`REPLY_SKIP`, `ANNOUNCE_SKIP`).

상세: [세션 도구](https://docs.openclaw.ai/concepts/session-tool)

## 스킬 레지스트리 (ClawHub)

ClawHub는 최소한의 스킬 레지스트리입니다. ClawHub가 활성화되면 에이전트가 자동으로 스킬을 검색하고 필요에 따라 새로운 스킬을 가져올 수 있습니다.

[ClawHub](https://clawhub.com)

## 채팅 명령

WhatsApp/Telegram/Slack/Google Chat/Microsoft Teams/WebChat에서 보내세요 (그룹 명령은 소유자 전용):

- `/status` — 간결한 세션 상태 (모델 + 토큰, 가능한 경우 비용)
- `/new` 또는 `/reset` — 세션 리셋
- `/compact` — 세션 컨텍스트 압축 (요약)
- `/think <level>` — off|minimal|low|medium|high|xhigh (GPT-5.2 + Codex 모델만)
- `/verbose on|off`
- `/usage off|tokens|full` — 응답별 사용량 푸터
- `/restart` — 게이트웨이 재시작 (그룹에서는 소유자 전용)
- `/activation mention|always` — 그룹 활성화 토글 (그룹 전용)

## 앱 (선택 사항)

게이트웨이만으로도 훌륭한 경험을 제공합니다. 모든 앱은 선택 사항이며 추가 기능을 제공합니다.

컴패니언 앱을 빌드/실행할 계획이라면 아래 플랫폼 런북을 따르세요.

### macOS (OpenClaw.app) (선택 사항)

- 게이트웨이 및 상태를 위한 메뉴 바 제어.
- Voice Wake + 푸시 투 토크 오버레이.
- WebChat + 디버그 도구.
- SSH를 통한 원격 게이트웨이 제어.

참고: macOS 권한이 재빌드 간에 유지되려면 서명된 빌드가 필요합니다 (`docs/mac/permissions.md` 참조).

### iOS 노드 (선택 사항)

- Bridge를 통해 노드로 페어링.
- 음성 트리거 포워딩 + Canvas 표면.
- `openclaw nodes ...`로 제어.

런북: [iOS 연결](https://docs.openclaw.ai/platforms/ios).

### Android 노드 (선택 사항)

- iOS와 동일한 Bridge + 페어링 흐름으로 페어링.
- Canvas, 카메라, 화면 캡처 명령을 노출합니다.
- 런북: [Android 연결](https://docs.openclaw.ai/platforms/android).

## 에이전트 워크스페이스 + 스킬

- 워크스페이스 루트: `~/.openclaw/workspace` (`agents.defaults.workspace`로 설정 가능).
- 주입되는 프롬프트 파일: `AGENTS.md`, `SOUL.md`, `TOOLS.md`.
- 스킬: `~/.openclaw/workspace/skills/<skill>/SKILL.md`.

## 설정

최소 `~/.openclaw/openclaw.json` (모델 + 기본값):

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-6",
  },
}
```

[전체 설정 참조 (모든 키 + 예시).](https://docs.openclaw.ai/gateway/configuration)

## 보안 모델 (중요)

- **기본값:** 도구는 **main** 세션에 대해 호스트에서 실행되므로 혼자 사용할 때 에이전트가 전체 접근 권한을 갖습니다.
- **그룹/채널 안전:** `agents.defaults.sandbox.mode: "non-main"`을 설정하여 **비main 세션**(그룹/채널)을 세션별 Docker 샌드박스 내에서 실행합니다; 해당 세션에서 bash는 Docker에서 실행됩니다.
- **샌드박스 기본값:** 허용 목록 `bash`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`; 거부 목록 `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`.

상세: [보안 가이드](https://docs.openclaw.ai/gateway/security) · [Docker + 샌드박싱](https://docs.openclaw.ai/install/docker) · [샌드박스 설정](https://docs.openclaw.ai/gateway/configuration)

### [WhatsApp](https://docs.openclaw.ai/channels/whatsapp)

- 디바이스 연결: `pnpm openclaw channels login` (`~/.openclaw/credentials`에 자격 증명 저장).
- `channels.whatsapp.allowFrom`으로 어시스턴트와 대화할 수 있는 사람을 허용 목록에 추가.
- `channels.whatsapp.groups`가 설정되면 그룹 허용 목록이 됩니다; 모두 허용하려면 `"*"`를 포함하세요.

### [Telegram](https://docs.openclaw.ai/channels/telegram)

- `TELEGRAM_BOT_TOKEN` 또는 `channels.telegram.botToken`을 설정합니다 (env가 우선).
- 선택 사항: `channels.telegram.groups` 설정 (`channels.telegram.groups."*".requireMention` 포함); 설정되면 그룹 허용 목록이 됩니다 (모두 허용하려면 `"*"` 포함). 또한 필요에 따라 `channels.telegram.allowFrom` 또는 `channels.telegram.webhookUrl` + `channels.telegram.webhookSecret`.

```json5
{
  channels: {
    telegram: {
      botToken: "123456:ABCDEF",
    },
  },
}
```

### [Slack](https://docs.openclaw.ai/channels/slack)

- `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` (또는 `channels.slack.botToken` + `channels.slack.appToken`)을 설정합니다.

### [Discord](https://docs.openclaw.ai/channels/discord)

- `DISCORD_BOT_TOKEN` 또는 `channels.discord.token`을 설정합니다 (env가 우선).
- 선택 사항: `commands.native`, `commands.text`, 또는 `commands.useAccessGroups`, 그리고 필요에 따라 `channels.discord.allowFrom`, `channels.discord.guilds`, 또는 `channels.discord.mediaMaxMb` 설정.

```json5
{
  channels: {
    discord: {
      token: "1234abcd",
    },
  },
}
```

### [Signal](https://docs.openclaw.ai/channels/signal)

- `signal-cli`와 `channels.signal` 설정 섹션이 필요합니다.

### [BlueBubbles (iMessage)](https://docs.openclaw.ai/channels/bluebubbles)

- **권장** iMessage 통합.
- `channels.bluebubbles.serverUrl` + `channels.bluebubbles.password` 및 웹훅(`channels.bluebubbles.webhookPath`)을 설정합니다.
- BlueBubbles 서버는 macOS에서 실행됩니다; 게이트웨이는 macOS 또는 다른 곳에서 실행할 수 있습니다.

### [iMessage (레거시)](https://docs.openclaw.ai/channels/imessage)

- `imsg`를 통한 레거시 macOS 전용 통합 (Messages에 로그인되어 있어야 함).
- `channels.imessage.groups`가 설정되면 그룹 허용 목록이 됩니다; 모두 허용하려면 `"*"`를 포함하세요.

### [Microsoft Teams](https://docs.openclaw.ai/channels/msteams)

- Teams 앱 + Bot Framework를 설정한 다음 `msteams` 설정 섹션을 추가합니다.
- `msteams.allowFrom`으로 대화할 수 있는 사람을 허용 목록에 추가; `msteams.groupAllowFrom` 또는 `msteams.groupPolicy: "open"`으로 그룹 접근.

### [WebChat](https://docs.openclaw.ai/web/webchat)

- 게이트웨이 WebSocket을 사용합니다; 별도의 WebChat 포트/설정 없음.

브라우저 제어 (선택 사항):

```json5
{
  browser: {
    enabled: true,
    color: "#FF4500",
  },
}
```

## 문서

온보딩 흐름을 지나 더 깊은 참조가 필요할 때 사용하세요.

- [문서 인덱스에서 탐색과 "어디에 무엇이 있는지"를 시작하세요.](https://docs.openclaw.ai)
- [게이트웨이 + 프로토콜 모델의 아키텍처 개요를 읽으세요.](https://docs.openclaw.ai/concepts/architecture)
- [모든 키와 예시가 필요할 때 전체 설정 참조를 사용하세요.](https://docs.openclaw.ai/gateway/configuration)
- [운영 런북으로 게이트웨이를 교과서대로 실행하세요.](https://docs.openclaw.ai/gateway)
- [Control UI/웹 표면이 어떻게 작동하고 안전하게 노출하는 방법을 알아보세요.](https://docs.openclaw.ai/web)
- [SSH 터널 또는 tailnet을 통한 원격 접근을 이해하세요.](https://docs.openclaw.ai/gateway/remote)
- [가이드 설정을 위한 온보딩 마법사 흐름을 따르세요.](https://docs.openclaw.ai/start/wizard)
- [웹훅 표면을 통해 외부 트리거를 연결하세요.](https://docs.openclaw.ai/automation/webhook)
- [Gmail Pub/Sub 트리거를 설정하세요.](https://docs.openclaw.ai/automation/gmail-pubsub)
- [macOS 메뉴 바 컴패니언 세부 사항을 알아보세요.](https://docs.openclaw.ai/platforms/mac/menu-bar)
- [플랫폼 가이드: Windows (WSL2)](https://docs.openclaw.ai/platforms/windows), [Linux](https://docs.openclaw.ai/platforms/linux), [macOS](https://docs.openclaw.ai/platforms/macos), [iOS](https://docs.openclaw.ai/platforms/ios), [Android](https://docs.openclaw.ai/platforms/android)
- [문제 해결 가이드로 일반적인 실패를 디버그하세요.](https://docs.openclaw.ai/channels/troubleshooting)
- [무엇이든 노출하기 전에 보안 지침을 검토하세요.](https://docs.openclaw.ai/gateway/security)

## 고급 문서 (검색 + 제어)

- [검색 + 전송](https://docs.openclaw.ai/gateway/discovery)
- [Bonjour/mDNS](https://docs.openclaw.ai/gateway/bonjour)
- [게이트웨이 페어링](https://docs.openclaw.ai/gateway/pairing)
- [원격 게이트웨이 README](https://docs.openclaw.ai/gateway/remote-gateway-readme)
- [Control UI](https://docs.openclaw.ai/web/control-ui)
- [대시보드](https://docs.openclaw.ai/web/dashboard)

## 운영 및 문제 해결

- [헬스 체크](https://docs.openclaw.ai/gateway/health)
- [게이트웨이 잠금](https://docs.openclaw.ai/gateway/gateway-lock)
- [백그라운드 프로세스](https://docs.openclaw.ai/gateway/background-process)
- [브라우저 문제 해결 (Linux)](https://docs.openclaw.ai/tools/browser-linux-troubleshooting)
- [로깅](https://docs.openclaw.ai/logging)

## 심층 분석

- [에이전트 루프](https://docs.openclaw.ai/concepts/agent-loop)
- [프레즌스](https://docs.openclaw.ai/concepts/presence)
- [TypeBox 스키마](https://docs.openclaw.ai/concepts/typebox)
- [RPC 어댑터](https://docs.openclaw.ai/reference/rpc)
- [큐](https://docs.openclaw.ai/concepts/queue)

## 워크스페이스 및 스킬

- [스킬 설정](https://docs.openclaw.ai/tools/skills-config)
- [기본 AGENTS](https://docs.openclaw.ai/reference/AGENTS.default)
- [템플릿: AGENTS](https://docs.openclaw.ai/reference/templates/AGENTS)
- [템플릿: BOOTSTRAP](https://docs.openclaw.ai/reference/templates/BOOTSTRAP)
- [템플릿: IDENTITY](https://docs.openclaw.ai/reference/templates/IDENTITY)
- [템플릿: SOUL](https://docs.openclaw.ai/reference/templates/SOUL)
- [템플릿: TOOLS](https://docs.openclaw.ai/reference/templates/TOOLS)
- [템플릿: USER](https://docs.openclaw.ai/reference/templates/USER)

## 플랫폼 내부

- [macOS 개발 설정](https://docs.openclaw.ai/platforms/mac/dev-setup)
- [macOS 메뉴 바](https://docs.openclaw.ai/platforms/mac/menu-bar)
- [macOS voice wake](https://docs.openclaw.ai/platforms/mac/voicewake)
- [iOS 노드](https://docs.openclaw.ai/platforms/ios)
- [Android 노드](https://docs.openclaw.ai/platforms/android)
- [Windows (WSL2)](https://docs.openclaw.ai/platforms/windows)
- [Linux 앱](https://docs.openclaw.ai/platforms/linux)

## 이메일 훅 (Gmail)

- [docs.openclaw.ai/gmail-pubsub](https://docs.openclaw.ai/automation/gmail-pubsub)

## Molty

OpenClaw는 우주 랍스터 AI 어시스턴트 **Molty**를 위해 만들어졌습니다.
Peter Steinberger와 커뮤니티가 함께 만들었습니다.

- [openclaw.ai](https://openclaw.ai)
- [soul.md](https://soul.md)
- [steipete.me](https://steipete.me)
- [@openclaw](https://x.com/openclaw)

## 커뮤니티

가이드라인, 관리자, PR 제출 방법은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.
AI/바이브 코딩 PR을 환영합니다!

[Mario Zechner](https://mariozechner.at/)의 지원과 [pi-mono](https://github.com/badlogic/pi-mono)에 특별한 감사를 드립니다.
Adam Doppelt의 lobster.bot에 특별한 감사를 드립니다.

모든 clawtributor에게 감사드립니다:

<p align="left">
  <a href="https://github.com/steipete"><img src="https://avatars.githubusercontent.com/u/58493?v=4&s=48" width="48" height="48" alt="steipete" title="steipete"/></a> <a href="https://github.com/sktbrd"><img src="https://avatars.githubusercontent.com/u/116202536?v=4&s=48" width="48" height="48" alt="sktbrd" title="sktbrd"/></a> <a href="https://github.com/cpojer"><img src="https://avatars.githubusercontent.com/u/13352?v=4&s=48" width="48" height="48" alt="cpojer" title="cpojer"/></a> <a href="https://github.com/joshp123"><img src="https://avatars.githubusercontent.com/u/1497361?v=4&s=48" width="48" height="48" alt="joshp123" title="joshp123"/></a> <a href="https://github.com/mbelinky"><img src="https://avatars.githubusercontent.com/u/132747814?v=4&s=48" width="48" height="48" alt="Mariano Belinky" title="Mariano Belinky"/></a> <a href="https://github.com/Takhoffman"><img src="https://avatars.githubusercontent.com/u/781889?v=4&s=48" width="48" height="48" alt="Takhoffman" title="Takhoffman"/></a> <a href="https://github.com/sebslight"><img src="https://avatars.githubusercontent.com/u/19554889?v=4&s=48" width="48" height="48" alt="sebslight" title="sebslight"/></a> <a href="https://github.com/tyler6204"><img src="https://avatars.githubusercontent.com/u/64381258?v=4&s=48" width="48" height="48" alt="tyler6204" title="tyler6204"/></a> <a href="https://github.com/quotentiroler"><img src="https://avatars.githubusercontent.com/u/40643627?v=4&s=48" width="48" height="48" alt="quotentiroler" title="quotentiroler"/></a> <a href="https://github.com/VeriteIgiraneza"><img src="https://avatars.githubusercontent.com/u/69280208?v=4&s=48" width="48" height="48" alt="Verite Igiraneza" title="Verite Igiraneza"/></a>
</p>

(전체 기여자 목록은 [영문 README](README.md)를 참조하세요)
