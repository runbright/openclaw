---
summary: "OpenClaw를 위한 고급 설정 및 개발 워크플로"
read_when:
  - 새로운 머신에서 설정을 시작할 때
  - 개인 설정을 유지하면서 최신 기능을 사용하고 싶을 때
title: "설정"
---

# 설정

<Note>
처음 설정하시는 경우 [시작하기](/start/getting-started)부터 읽어보세요.
위저드에 대한 상세 내용은 [온보딩 위저드](/start/wizard)를 참고하세요.
</Note>

최종 업데이트: 2026-01-01

## 요약 (TL;DR)

- **커스터마이징은 레포지토리 외부에서 관리됩니다:** `~/.openclaw/workspace` (워크스페이스) + `~/.openclaw/openclaw.json` (구성 설정).
- **안정적인 워크플로:** macOS 앱을 설치하고, 앱에 내장된 게이트웨이를 실행하세요.
- **최신(Bleeding Edge) 워크플로:** `pnpm gateway:watch`를 통해 게이트웨이를 직접 실행하고, macOS 앱을 로컬 모드로 연결하세요.

## 요구 사항 (소스 설치 시)

- Node `>=22`
- `pnpm`
- Docker (선택 사항; 컨테이너 기반 설정/E2E 테스트용 — [Docker](/install/docker) 참고)

## 커스터마이징 전략 (업데이트 시 영향 최소화)

"완벽한 개인화"와 "쉬운 업데이트"를 동시에 원한다면, 커스텀 설정을 다음 위치에 보관하세요:

- **구성 설정:** `~/.openclaw/openclaw.json` (JSON/JSON5 형식)
- **워크스페이스:** `~/.openclaw/workspace` (스킬, 프롬프트, 메모리; 개인 Git 레포지토리로 관리하는 것을 권장)

초기 부트스트랩:

```bash
openclaw setup
```

레포지토리 내부에서 실행할 때는 로컬 CLI 엔트리를 사용하세요:

```bash
openclaw setup
```

글로벌 설치가 되어 있지 않다면 `pnpm openclaw setup`을 통해 실행할 수 있습니다.

## 이 레포지토리에서 게이트웨이 실행하기

`pnpm build` 이후에 패키징된 CLI를 직접 실행할 수 있습니다:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## 안정적인 워크플로 (macOS 앱 우선)

1. **OpenClaw.app**을 설치하고 실행합니다 (메뉴 바 앱).
2. 온보딩/권한 체크리스트(TCC 요청 등)를 완료합니다.
3. 게이트웨이가 **Local** 모드이고 실행 중인지 확인합니다 (앱이 자동으로 관리합니다).
4. 채널을 연결합니다 (예: WhatsApp):

```bash
openclaw channels login
```

5. 상태 점검:

```bash
openclaw health
```

사용 중인 빌드에서 온보딩을 사용할 수 없는 경우:

- `openclaw setup` 실행 후 `openclaw channels login`을 수행하고, 게이트웨이를 수동으로 시작(`openclaw gateway`)하세요.

## 최신 워크플로 (터미널에서 게이트웨이 실행)

목표: TypeScript 게이트웨이를 수정하면서 실시간 반영(hot reload)을 적용하고, macOS 앱 UI를 연결된 상태로 유지하는 것입니다.

### 0) (선택 사항) macOS 앱도 소스에서 실행하기

macOS 앱도 최신 소스로 실행하려면:

```bash
./scripts/restart-mac.sh
```

### 1) 개발용 게이트웨이 시작

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch`는 감시 모드에서 게이트웨이를 실행하며 TypeScript 파일이 변경될 때마다 다시 로드합니다.

### 2) macOS 앱이 실행 중인 게이트웨이를 바라보도록 설정

**OpenClaw.app**에서:

- Connection Mode: **Local**
  앱이 구성된 포트에서 실행 중인 게이트웨이에 연결됩니다.

### 3) 확인

- 앱 내 게이트웨이 상태가 **“Using existing gateway …”**로 표시되어야 합니다.
- 또는 CLI를 통해 확인:

```bash
openclaw health
```

### 흔한 실수 (Footguns)

- **잘못된 포트:** 게이트웨이 WebSocket 기본값은 `ws://127.0.0.1:18789`입니다. 앱과 CLI가 동일한 포트를 사용하는지 확인하세요.
- **상태 데이터 저장 위치:**
  - 인증 정보(Credentials): `~/.openclaw/credentials/`
  - 세션: `~/.openclaw/agents/<agentId>/sessions/`
  - 로그: `/tmp/openclaw/`

## 인증 정보 저장소 맵

인증 문제를 디버깅하거나 백업 대상을 결정할 때 참고하세요:

- **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Telegram 봇 토큰**: 설정/환경 변수 또는 `channels.telegram.tokenFile`
- **Discord 봇 토큰**: 설정/환경 변수 (토큰 파일 방식은 아직 지원되지 않음)
- **Slack 토큰**: 설정/환경 변수 (`channels.slack.*`)
- **페어링 허용 목록**: `~/.openclaw/credentials/<channel>-allowFrom.json`
- **모델 인증 프로필**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **레거시 OAuth 가져오기**: `~/.openclaw/credentials/oauth.json`
  상세 내용: [보안](/gateway/security#credential-storage-map).

## 업데이트 (개인 설정 유지하기)

- `~/.openclaw/workspace` 및 `~/.openclaw/` 폴더를 "개인 영역"으로 유지하세요. `openclaw` 레포지토리에 개인용 프롬프트나 설정을 넣지 마세요.
- 소스 업데이트: `git pull` + `pnpm install` (lockfile이 변경된 경우)을 수행한 뒤 `pnpm gateway:watch`를 계속 사용하세요.

## Linux (systemd 사용자 서비스)

Linux에서는 systemd **사용자(user)** 서비스를 사용합니다. 기본적으로 systemd는 로그아웃이나 유휴 상태 시 사용자 서비스를 중지하여 게이트웨이가 종료될 수 있습니다. 온보딩 과정에서 링거링(lingering) 활성화를 시도하지만(sudo 요청이 있을 수 있음), 여전히 꺼져 있다면 다음을 실행하세요:

```bash
sudo loginctl enable-linger $USER
```

항상 켜져 있어야 하거나 다중 사용자가 사용하는 서버의 경우, 사용자 서비스 대신 **시스템(system)** 서비스를 고려해 보세요. 상세 내용은 [게이트웨이 운영 가이드](/gateway)의 systemd 참고 사항을 확인하세요.

## 관련 문서

- [게이트웨이 운영 가이드](/gateway) (플래그, 감독, 포트)
- [구성 설정](/gateway/configuration) (설정 스키마 및 예시)
- [Discord](/channels/discord) 및 [Telegram](/channels/telegram) (답장 태그 및 replyToMode 설정)
- [OpenClaw 어시스턴트 설정](/start/openclaw)
- [macOS 앱](/platforms/macos) (게이트웨이 생명 주기)
