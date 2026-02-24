---
summary: "Nix로 OpenClaw을 선언적으로 설치"
read_when:
  - 재현 가능하고 롤백 가능한 설치를 원할 때
  - 이미 Nix/NixOS/Home Manager를 사용 중일 때
  - 모든 것을 고정하고 선언적으로 관리하고 싶을 때
title: "Nix"
---

# Nix 설치

Nix로 OpenClaw을 실행하는 권장 방법은 **[nix-openclaw](https://github.com/openclaw/nix-openclaw)**을 사용하는 것입니다 — 배터리가 포함된 Home Manager 모듈입니다.

## 빠른 시작

이것을 AI 에이전트 (Claude, Cursor 등)에 붙여넣으세요:

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 전체 가이드: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw 저장소가 Nix 설치의 기준 문서입니다. 이 페이지는 간략한 개요입니다.

## 제공되는 기능

- 게이트웨이 + macOS 앱 + 도구 (whisper, spotify, cameras) — 모두 고정
- 재부팅에서도 유지되는 Launchd 서비스
- 선언적 구성을 갖춘 플러그인 시스템
- 즉시 롤백: `home-manager switch --rollback`

---

## Nix 모드 런타임 동작

`OPENCLAW_NIX_MODE=1`이 설정되면 (nix-openclaw에서 자동):

OpenClaw은 구성을 결정적으로 만들고 자동 설치 흐름을 비활성화하는 **Nix 모드**를 지원합니다.
다음을 내보내서 활성화하세요:

```bash
OPENCLAW_NIX_MODE=1
```

macOS에서 GUI 앱은 셸 환경 변수를 자동으로 상속하지 않습니다. defaults를 통해서도
Nix 모드를 활성화할 수 있습니다:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### 설정 + 상태 경로

OpenClaw은 `OPENCLAW_CONFIG_PATH`에서 JSON5 설정을 읽고 `OPENCLAW_STATE_DIR`에 가변 데이터를 저장합니다.
필요 시 `OPENCLAW_HOME`을 설정하여 내부 경로 해석에 사용되는 기본 홈 디렉토리를 제어할 수도 있습니다.

- `OPENCLAW_HOME` (기본 우선순위: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (기본: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (기본: `$OPENCLAW_STATE_DIR/openclaw.json`)

Nix에서 실행할 때 런타임 상태와 설정이 불변 스토어 밖에 유지되도록
이들을 Nix 관리 위치로 명시적으로 설정하세요.

### Nix 모드에서의 런타임 동작

- 자동 설치 및 자기 변이 흐름이 비활성화됨
- 누락된 의존성은 Nix 전용 해결 방법 메시지를 표시
- UI에 읽기 전용 Nix 모드 배너가 표시됨

## 패키징 참고 (macOS)

macOS 패키징 흐름은 다음 위치의 안정적인 Info.plist 템플릿을 기대합니다:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh)는 이 템플릿을 앱 번들에 복사하고 동적 필드
(번들 ID, 버전/빌드, Git SHA, Sparkle 키)를 패치합니다. 이렇게 하면 SwiftPM
패키징과 Nix 빌드 (전체 Xcode 도구 체인에 의존하지 않음)에서 plist가 결정적으로 유지됩니다.

## 관련 문서

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — 전체 설정 가이드
- [마법사](/start/wizard) — 비 Nix CLI 설정
- [Docker](/install/docker) — 컨테이너화된 설정
