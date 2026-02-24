---
summary: "macOS에서의 게이트웨이 런타임 (외부 launchd 서비스)"
read_when:
  - OpenClaw.app 패키징
  - macOS 게이트웨이 launchd 서비스 디버깅
  - macOS용 게이트웨이 CLI 설치
title: "macOS에서의 게이트웨이"
---

# macOS에서의 게이트웨이 (외부 launchd)

OpenClaw.app은 더 이상 Node/Bun 또는 게이트웨이 런타임을 번들로 포함하지 않습니다. macOS 앱은 **외부** `openclaw` CLI 설치를 기대하며, 게이트웨이를 자식 프로세스로 생성하지 않고, 게이트웨이를 계속 실행하기 위해 사용자별 launchd 서비스를 관리합니다 (또는 이미 실행 중인 로컬 게이트웨이가 있으면 연결합니다).

## CLI 설치 (로컬 모드에 필요)

Mac에 Node 22 이상이 필요하며, `openclaw`를 전역으로 설치하세요:

```bash
npm install -g openclaw@<version>
```

macOS 앱의 **Install CLI** 버튼은 npm/pnpm을 통해 동일한 흐름을 실행합니다 (bun은 게이트웨이 런타임에 권장하지 않음).

## Launchd (LaunchAgent로서의 게이트웨이)

레이블:

- `bot.molt.gateway` (또는 `bot.molt.<profile>`; 레거시 `com.openclaw.*`가 남아 있을 수 있음)

Plist 위치 (사용자별):

- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  (또는 `~/Library/LaunchAgents/bot.molt.<profile>.plist`)

관리자:

- macOS 앱이 로컬 모드에서 LaunchAgent 설치/업데이트를 담당합니다.
- CLI로도 설치할 수 있습니다: `openclaw gateway install`.

동작:

- "OpenClaw Active"로 LaunchAgent를 활성화/비활성화합니다.
- 앱을 종료해도 게이트웨이는 **중지되지 않습니다** (launchd가 유지).
- 구성된 포트에서 이미 게이트웨이가 실행 중이면, 앱은 새 게이트웨이를 시작하는 대신 기존 게이트웨이에 연결합니다.

로깅:

- launchd stdout/err: `/tmp/openclaw/openclaw-gateway.log`

## 버전 호환성

macOS 앱은 게이트웨이 버전을 자체 버전과 비교합니다. 호환되지 않으면 앱 버전에 맞게 전역 CLI를 업데이트하세요.

## 스모크 체크

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

그 다음:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
