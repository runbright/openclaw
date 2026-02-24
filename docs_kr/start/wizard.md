---
summary: "CLI 온보딩 위저드: 게이트웨이, 워크스페이스, 채널 및 스킬을 위한 가이드 설정"
read_when:
  - 온보딩 위저드를 실행하거나 구성할 때
  - 새로운 머신에서 설정을 시작할 때
title: "온보딩 위저드 (CLI)"
sidebarTitle: "온보딩: CLI"
---

# 온보딩 위저드 (CLI)

온보딩 위저드는 macOS, Linux 또는 Windows(WSL2 경유; 강력 권장)에서 OpenClaw를 설정하는 **권장** 방법입니다.
가이드에 따라 로컬 게이트웨이 또는 원격 게이트웨이 연결, 채널, 스킬 및 워크스페이스 기본값 설정을 한 번에 완료할 수 있습니다.

```bash
openclaw onboard
```

<Info>
가장 빠른 첫 채팅 방법: 채널 설정 없이 제어 UI를 여는 것입니다.
`openclaw dashboard`를 실행하고 브라우저에서 채팅하세요. 관련 문서: [대시보드](/web/dashboard).
</Info>

나중에 다시 구성하려면:

```bash
openclaw configure
openclaw agents add <이름>
```

<Note>
`--json` 플래그가 비대화형(non-interactive) 모드를 의미하지는 않습니다. 스크립트에서 사용하려면 `--non-interactive`를 사용하세요.
</Note>

<Tip>
에이전트가 `web_search` 기능을 사용할 수 있도록 Brave Search API 키를 설정하는 것을 권장합니다 (`web_fetch`는 키 없이도 작동합니다). 가장 쉬운 방법은 `openclaw configure --section web`을 실행하는 것이며, 이는 `tools.web.search.apiKey`에 저장됩니다. 관련 문서: [웹 도구](/tools/web).
</Tip>

## 빠른 시작 vs 고급 설정

위저드는 **빠른 시작(QuickStart)** (기본값) 또는 **고급 설정(Advanced)** (전체 제어) 중 하나를 선택하며 시작합니다.

<Tabs>
  <Tab title="빠른 시작 (기본값)">
    - 로컬 게이트웨이 (루프백)
    - 워크스페이스 기본값 (또는 기존 워크스페이스)
    - 게이트웨이 포트 **18789**
    - 게이트웨이 인증 **토큰** (루프백이라도 자동 생성됨)
    - DM 격리 기본값: 로컬 온보딩 시 설정되어 있지 않으면 `session.dmScope: "per-channel-peer"`를 작성합니다. 상세 내용: [CLI 온보딩 참조](/start/wizard-cli-reference#outputs-and-internals)
    - Tailscale 노출 **비활성**
    - Telegram 및 WhatsApp DM의 기본값은 **allowlist**입니다 (전화번호 입력을 요청받게 됩니다).
  </Tab>
  <Tab title="고급 설정 (전체 제어)">
    - 모든 단계(모드, 워크스페이스, 게이트웨이, 채널, 데몬, 스킬)를 직접 설정합니다.
  </Tab>
</Tabs>

## 위저드가 설정하는 항목

**로컬 모드 (기본값)**는 다음 단계를 안내합니다:

1. **모델/인증** — Anthropic API 키 (권장), OpenAI 또는 커스텀 프로바이더 (OpenAI 호환, Anthropic 호환 또는 자동 감지). 기본 모델을 선택합니다.
2. **워크스페이스** — 에이전트 파일 위치 (기본값 `~/.openclaw/workspace`). 초기 부트스트랩 파일을 생성합니다.
3. **게이트웨이** — 포트, 바인딩 주소, 인증 모드, Tailscale 노출 여부.
4. **채널** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles 또는 iMessage.
5. **데몬** — LaunchAgent (macOS) 또는 systemd 사용자 유닛 (Linux/WSL2)을 설치합니다.
6. **상태 확인** — 게이트웨이를 시작하고 실행 여부를 확인합니다.
7. **스킬** — 권장 스킬 및 선택적 의존성을 설치합니다.

<Note>
위저드를 다시 실행해도 **초기화(Reset)**를 명시적으로 선택하거나 `--reset` 플래그를 전달하지 않는 한 데이터가 삭제되지 않습니다.
설정이 유효하지 않거나 오래된 키가 포함된 경우, 위저드는 먼저 `openclaw doctor`를 실행하도록 요청합니다.
</Note>

**원격 모드**는 로컬 클라이언트가 다른 곳에 있는 게이트웨이에 연결할 수 있도록만 설정합니다.
원격 호스트에는 아무 것도 설치하거나 변경하지 않습니다.

## 다른 에이전트 추가

`openclaw agents add <이름>`을 사용하여 자체 워크스페이스, 세션 및 인증 프로필을 가진 별도의 에이전트를 만듭니다. `--workspace` 없이 실행하면 위저드가 시작됩니다.

설정되는 항목:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

참고 사항:

- 기본 워크스페이스는 `~/.openclaw/workspace-<agentId>` 경로를 따릅니다.
- 인바운드 메시지 라우팅을 위해 `bindings`를 추가합니다 (위저드에서 수행 가능).
- 비대화형 플래그: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## 전체 참조

단계별 상세 분석, 비대화형 스크립팅, Signal 설정, RPC API 및 위저드가 작성하는 전체 설정 필드 목록은 [위저드 참조 문서](/reference/wizard)를 확인하세요.

## 관련 문서

- CLI 명령 참조: [`openclaw onboard`](/cli/onboard)
- 온보딩 개요: [온보딩 개요](/start/onboarding-overview)
- macOS 앱 온보딩: [온보딩](/start/onboarding)
- 에이전트 첫 실행 리추얼: [에이전트 부트스트래핑](/start/bootstrapping)
