---
summary: "원격 OpenClaw 게이트웨이를 SSH를 통해 제어하는 macOS 앱 흐름"
read_when:
  - 원격 macOS 제어 설정 또는 디버깅
title: "원격 제어"
---

# 원격 OpenClaw (macOS ⇄ 원격 호스트)

이 흐름은 macOS 앱이 다른 호스트 (데스크톱/서버)에서 실행 중인 OpenClaw 게이트웨이의 완전한 원격 제어 역할을 할 수 있게 합니다. 앱의 **Remote over SSH** (원격 실행) 기능입니다. 상태 확인, Voice Wake 전달 및 Web Chat 등 모든 기능이 _Settings → General_의 동일한 원격 SSH 구성을 재사용합니다.

## 모드

- **Local (이 Mac)**: 모든 것이 노트북에서 실행됩니다. SSH가 관여하지 않습니다.
- **Remote over SSH (기본값)**: OpenClaw 명령이 원격 호스트에서 실행됩니다. macOS 앱은 `-o BatchMode`와 선택한 ID/키 및 로컬 포트 포워딩으로 SSH 연결을 엽니다.
- **Remote direct (ws/wss)**: SSH 터널 없음. macOS 앱이 게이트웨이 URL에 직접 연결합니다 (예: Tailscale Serve 또는 공개 HTTPS 리버스 프록시 사용).

## 원격 전송

원격 모드는 두 가지 전송을 지원합니다:

- **SSH 터널** (기본값): `ssh -N -L ...`을 사용하여 게이트웨이 포트를 localhost로 포워딩합니다. 터널이 loopback이므로 게이트웨이는 노드의 IP를 `127.0.0.1`로 인식합니다.
- **Direct (ws/wss)**: 게이트웨이 URL에 직접 연결합니다. 게이트웨이가 실제 클라이언트 IP를 인식합니다.

## 원격 호스트의 사전 요구사항

1. Node + pnpm을 설치하고 OpenClaw CLI를 빌드/설치하세요 (`pnpm install && pnpm build && pnpm link --global`).
2. 비대화형 쉘에서 `openclaw`이 PATH에 있는지 확인하세요 (필요하면 `/usr/local/bin` 또는 `/opt/homebrew/bin`에 심볼릭 링크).
3. 키 인증으로 SSH를 여세요. LAN 외부에서 안정적인 접근을 위해 **Tailscale** IP를 권장합니다.

## macOS 앱 설정

1. _Settings → General_을 여세요.
2. **OpenClaw runs**에서 **Remote over SSH**를 선택하고 설정하세요:
   - **Transport**: **SSH tunnel** 또는 **Direct (ws/wss)**.
   - **SSH target**: `user@host` (선택사항 `:port`).
     - 게이트웨이가 동일 LAN에 있고 Bonjour를 광고하면, 검색된 목록에서 선택하여 이 필드를 자동 입력합니다.
   - **Gateway URL** (Direct만): `wss://gateway.example.ts.net` (또는 로컬/LAN용 `ws://...`).
   - **Identity file** (고급): 키 경로.
   - **Project root** (고급): 명령에 사용되는 원격 체크아웃 경로.
   - **CLI path** (고급): 실행 가능한 `openclaw` 진입점/바이너리 경로 (선택사항, 광고 시 자동 입력).
3. **Test remote**을 누르세요. 성공하면 원격 `openclaw status --json`이 올바르게 실행됨을 나타냅니다. 실패는 보통 PATH/CLI 문제입니다; exit 127은 원격에서 CLI를 찾을 수 없음을 의미합니다.
4. 상태 확인과 Web Chat이 이제 이 SSH 터널을 통해 자동으로 실행됩니다.

## Web Chat

- **SSH 터널**: Web Chat이 포워딩된 WebSocket 제어 포트 (기본값 18789)를 통해 게이트웨이에 연결합니다.
- **Direct (ws/wss)**: Web Chat이 구성된 게이트웨이 URL에 직접 연결합니다.
- 별도의 WebChat HTTP 서버는 더 이상 없습니다.

## 권한

- 원격 호스트에는 로컬과 동일한 TCC 승인이 필요합니다 (자동화, 접근성, 화면 녹화, 마이크, 음성 인식, 알림). 해당 머신에서 온보딩을 실행하여 한 번 부여하세요.
- 노드는 `node.list` / `node.describe`를 통해 권한 상태를 광고하므로 에이전트가 사용 가능한 항목을 알 수 있습니다.

## 보안 참고사항

- 원격 호스트에서 loopback 바인딩을 선호하고 SSH 또는 Tailscale을 통해 연결하세요.
- SSH 터널링은 엄격한 호스트 키 확인을 사용합니다; 먼저 호스트 키를 신뢰하여 `~/.ssh/known_hosts`에 존재하도록 하세요.
- 게이트웨이를 비loopback 인터페이스에 바인딩하면 토큰/비밀번호 인증을 요구하세요.
- [보안](/gateway/security) 및 [Tailscale](/gateway/tailscale)을 참조하세요.

## WhatsApp 로그인 흐름 (원격)

- **원격 호스트에서** `openclaw channels login --verbose`를 실행하세요. 휴대폰의 WhatsApp으로 QR을 스캔하세요.
- 인증이 만료되면 해당 호스트에서 로그인을 다시 실행하세요. 상태 확인이 링크 문제를 표시합니다.

## 문제 해결

- **exit 127 / not found**: `openclaw`이 비로그인 쉘의 PATH에 없습니다. `/etc/paths`, 쉘 rc에 추가하거나 `/usr/local/bin`/`/opt/homebrew/bin`에 심볼릭 링크하세요.
- **상태 프로브 실패**: SSH 접근 가능성, PATH, Baileys가 로그인되어 있는지 (`openclaw status --json`) 확인하세요.
- **Web Chat 멈춤**: 원격 호스트에서 게이트웨이가 실행 중이고 포워딩된 포트가 게이트웨이 WS 포트와 일치하는지 확인하세요; UI는 정상적인 WS 연결이 필요합니다.
- **노드 IP가 127.0.0.1 표시**: SSH 터널에서 예상되는 동작입니다. 게이트웨이가 실제 클라이언트 IP를 인식하길 원하면 **Transport**를 **Direct (ws/wss)**로 전환하세요.
- **Voice Wake**: 원격 모드에서 트리거 문구가 자동으로 전달됩니다; 별도의 포워더가 필요하지 않습니다.

## 알림 소리

스크립트에서 `openclaw`과 `node.invoke`를 사용하여 알림별로 소리를 선택하세요. 예:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

앱에는 더 이상 글로벌 "기본 소리" 토글이 없습니다; 호출자가 요청별로 소리를 선택합니다 (또는 선택하지 않음).
