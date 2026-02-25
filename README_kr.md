# 🦞 OpenClaw — 개인용 AI 어시스턴트

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

**OpenClaw**는 사용자의 기기에서 직접 실행되는 *개인용 AI 어시스턴트*입니다.
현재 사용 중인 다양한 채널(WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, WebChat)뿐만 아니라 BlueBubbles, Matrix, Zalo, Zalo Personal 같은 확장 채널에서도 답변을 제공합니다. macOS/iOS/Android에서 음성 대화가 가능하며, 사용자가 제어할 수 있는 라이브 캔버스(Canvas)를 렌더링할 수 있습니다. 게이트웨이는 제어 계층일 뿐, 제품의 핵심은 바로 이 어시스턴트입니다.

속도가 빠르고, 로컬 중심이며, 항상 켜져 있는 나만의 개인용 어시스턴트를 원하신다면 OpenClaw가 정답입니다.

[웹사이트](https://openclaw.ai) · [문서](https://docs.openclaw.ai) · [비전](VISION.md) · [DeepWiki](https://deepwiki.com/openclaw/openclaw) · [시작하기](https://docs.openclaw.ai/start/getting-started) · [업데이트](https://docs.openclaw.ai/install/updating) · [쇼케이스](https://docs.openclaw.ai/start/showcase) · [FAQ](https://docs.openclaw.ai/start/faq) · [마법사](https://docs.openclaw.ai/start/wizard) · [Docker](https://docs.openclaw.ai/install/docker) · [Discord](https://discord.gg/clawd)

권장 설정: 터미널에서 온보딩 마법사(`openclaw onboard`)를 실행하세요.
마법사가 게이트웨이, 워크스페이스, 채널 및 스킬 설정을 단계별로 안내합니다. 이 CLI 마법사는 **macOS, Linux, Windows(WSL2 권장)**에서 작동하며 가장 권장되는 설치 경로입니다.

새로 설치하시나요? 여기서 시작하세요: [시작하기(영문)](https://docs.openclaw.ai/start/getting-started)

## 모델 (선택 및 인증)
- 모델 설정 및 CLI 상세: [모델 가이드(영문)](https://docs.openclaw.ai/concepts/models)
- 인증 프로필 로테이션 및 장애 복구: [모델 페일오버(영문)](https://docs.openclaw.ai/concepts/model-failover)

## 설치 (권장)
사용 환경: **Node ≥22**.

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```
마법사가 게이트웨이 데몬(launchd/systemd 서비스)을 설치하여 백그라운드에서 항상 실행되도록 합니다.

## 빠른 시작 (요약)
```bash
# 데몬 설치 및 설정
openclaw onboard --install-daemon

# 게이트웨이 실행
openclaw gateway --port 18789 --verbose

# 메시지 보내기
openclaw message send --to +1234567890 --message "OpenClaw에서 보낸 메시지입니다"

# 에이전트와 대화하기
openclaw agent --message "체크리스트 작성해줘" --thinking high
```

## 주요 기능 (Highlights)
- **로컬 우선 게이트웨이**: 세션, 채널, 도구, 이벤트를 위한 단일 제어 계층.
- **멀티 채널 지원**: 주요 메시징 앱 및 특수 플랫폼 완벽 지원.
- **멀티 에이전트 라우팅**: 채널/계정별로 에이전트를 격리하여 운영 가능.
- **음성 호출 및 대화 모드**: ElevenLabs를 활용한 macOS/iOS/Android 음성 지원.
- **라이브 캔버스**: 에이전트 기반의 시각적 워크스페이스.
- **최상급 도구 모음**: 브라우저 제어, 캔버스, 크론탭, 세션 관리 등.

## 작동 원리 (간략도)
```
메시징 앱 (WhatsApp / Telegram / Slack / Discord 등)
               │
               ▼
┌───────────────────────────────┐
│            게이트웨이         │
│         (제어 계층)           │
│     ws://127.0.0.1:18789      │
└──────────────┬────────────────┘
               │
               ├─ Pi 에이전트 (RPC)
               ├─ CLI (openclaw …)
               ├─ WebChat UI (웹 채팅)
               ├─ macOS 앱
               └─ iOS / Android 노드
```

---
상세한 문서와 가이드는 [공식 문서 사이트](https://docs.openclaw.ai)를 참조하세요.
기여 방법은 [CONTRIBUTING.md](CONTRIBUTING.md)에서 확인하실 수 있습니다.
---
