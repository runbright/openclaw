---
summary: "Gateway를 위한 브라우저 기반 제어 UI (채팅, 노드, 설정)"
read_when:
  - 브라우저에서 Gateway를 운영하려는 경우
  - SSH 터널 없이 Tailnet 접근을 원하는 경우
title: "제어 UI"
---

# 제어 UI (브라우저)

제어 UI는 Gateway가 제공하는 작은 **Vite + Lit** 싱글 페이지 앱입니다:

- 기본값: `http://<host>:18789/`
- 선택적 접두사: `gateway.controlUi.basePath` 설정 (예: `/openclaw`)

동일한 포트의 **Gateway WebSocket에 직접** 통신합니다.

## 빠른 열기 (로컬)

Gateway가 같은 컴퓨터에서 실행 중이면:

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (또는 [http://localhost:18789/](http://localhost:18789/))

페이지 로드에 실패하면 먼저 Gateway를 시작하세요: `openclaw gateway`.

인증은 WebSocket 핸드셰이크 중 다음을 통해 제공됩니다:

- `connect.params.auth.token`
- `connect.params.auth.password`
  대시보드 설정 패널에서 토큰을 저장할 수 있습니다; 비밀번호는 저장되지 않습니다.
  온보딩 마법사가 기본적으로 gateway 토큰을 생성하므로, 첫 번째 연결 시 여기에 붙여넣으세요.

## 기기 페어링 (첫 연결)

새 브라우저나 기기에서 제어 UI에 연결할 때, `gateway.auth.allowTailscale: true`로
동일한 Tailnet에 있더라도 Gateway는 **일회성 페어링 승인**을 요구합니다. 이는 무단 접근을
방지하기 위한 보안 조치입니다.

**표시되는 내용:** "disconnected (1008): pairing required"

**기기를 승인하려면:**

```bash
# 대기 중인 요청 목록
openclaw devices list

# 요청 ID로 승인
openclaw devices approve <requestId>
```

승인되면 기기가 기억되며, `openclaw devices revoke --device <id> --role <role>`로
철회하지 않는 한 재승인이 필요하지 않습니다. 토큰 순환 및 철회에 대해서는
[기기 CLI](/cli/devices)를 참조하세요.

**참고:**

- 로컬 연결 (`127.0.0.1`)은 자동 승인됩니다.
- 원격 연결 (LAN, Tailnet 등)은 명시적 승인이 필요합니다.
- 각 브라우저 프로필은 고유한 기기 ID를 생성하므로, 브라우저를 전환하거나
  브라우저 데이터를 지우면 재페어링이 필요합니다.

## 현재 가능한 기능

- Gateway WS를 통한 모델과 채팅 (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- 채팅에서 도구 호출 + 실시간 도구 출력 카드 스트리밍 (에이전트 이벤트)
- 채널: WhatsApp/Telegram/Discord/Slack + 플러그인 채널 (Mattermost 등) 상태 + QR 로그인 + 채널별 설정 (`channels.status`, `web.login.*`, `config.patch`)
- 인스턴스: 존재 목록 + 갱신 (`system-presence`)
- 세션: 목록 + 세션별 thinking/verbose 재정의 (`sessions.list`, `sessions.patch`)
- Cron 작업: 목록/추가/편집/실행/활성화/비활성화 + 실행 이력 (`cron.*`)
- 스킬: 상태, 활성화/비활성화, 설치, API 키 업데이트 (`skills.*`)
- 노드: 목록 + 기능 (`node.list`)
- Exec 승인: gateway 또는 노드 허용 목록 편집 + `exec host=gateway/node`에 대한 요청 정책 (`exec.approvals.*`)
- 설정: `~/.openclaw/openclaw.json` 보기/편집 (`config.get`, `config.set`)
- 설정: 유효성 검사를 통한 적용 + 재시작 (`config.apply`) 및 마지막 활성 세션 깨우기
- 설정 쓰기는 동시 편집을 덮어쓰는 것을 방지하는 base-hash 가드를 포함
- 설정 스키마 + 양식 렌더링 (`config.schema`, 플러그인 + 채널 스키마 포함); 원시 JSON 편집기도 사용 가능
- 디버그: 상태/건강/모델 스냅샷 + 이벤트 로그 + 수동 RPC 호출 (`status`, `health`, `models.list`)
- 로그: 필터/내보내기를 통한 gateway 파일 로그 실시간 테일 (`logs.tail`)
- 업데이트: 패키지/git 업데이트 실행 + 재시작 (`update.run`) 재시작 보고서 포함

Cron 작업 패널 참고:

- 격리된 작업의 경우 전달은 기본적으로 알림 요약입니다. 내부 전용 실행을 원하면 none으로 전환할 수 있습니다.
- 알림이 선택되면 채널/대상 필드가 나타납니다.
- 웹훅 모드는 `delivery.mode = "webhook"`을 사용하며 `delivery.to`가 유효한 HTTP(S) 웹훅 URL로 설정됩니다.
- 메인 세션 작업의 경우 웹훅 및 none 전달 모드를 사용할 수 있습니다.
- 고급 편집 컨트롤에는 실행 후 삭제, 에이전트 재정의 지우기, cron 정확/분산 옵션,
  에이전트 모델/thinking 재정의, 최선 노력 전달 토글이 포함됩니다.
- 양식 유효성 검사는 필드 수준 오류로 인라인입니다; 유효하지 않은 값은 수정될 때까지 저장 버튼을 비활성화합니다.
- 전용 bearer 토큰을 보내려면 `cron.webhookToken`을 설정하세요. 생략하면 인증 헤더 없이 웹훅이 전송됩니다.
- 폐기된 폴백: `notify: true`가 있는 저장된 레거시 작업은 마이그레이션될 때까지 여전히 `cron.webhook`을 사용할 수 있습니다.

## 채팅 동작

- `chat.send`는 **논블로킹**입니다: `{ runId, status: "started" }`로 즉시 확인하고 응답은 `chat` 이벤트를 통해 스트리밍됩니다.
- 동일한 `idempotencyKey`로 재전송하면 실행 중에는 `{ status: "in_flight" }`를, 완료 후에는 `{ status: "ok" }`를 반환합니다.
- `chat.history` 응답은 UI 안전을 위해 크기가 제한됩니다. 트랜스크립트 항목이 너무 크면 Gateway가 긴 텍스트 필드를 잘라내고, 무거운 메타데이터 블록을 생략하며, 초과 크기 메시지를 플레이스홀더 (`[chat.history omitted: message too large]`)로 대체할 수 있습니다.
- `chat.inject`는 세션 트랜스크립트에 어시스턴트 노트를 추가하고 UI 전용 업데이트를 위한 `chat` 이벤트를 브로드캐스트합니다 (에이전트 실행 없음, 채널 전달 없음).
- 중지:
  - **Stop** 클릭 (`chat.abort` 호출)
  - `/stop` 입력 (또는 `stop|esc|abort|wait|exit|interrupt`)으로 대역 외 중단
  - `chat.abort`는 해당 세션의 모든 활성 실행을 중단하기 위해 `{ sessionKey }`를 지원합니다 (`runId` 없음)
- 중단 부분 유지:
  - 실행이 중단되면 부분 어시스턴트 텍스트가 UI에 여전히 표시될 수 있습니다
  - Gateway는 버퍼된 출력이 있을 때 중단된 부분 어시스턴트 텍스트를 트랜스크립트 이력에 저장합니다
  - 저장된 항목에는 중단 메타데이터가 포함되어 트랜스크립트 소비자가 중단 부분과 정상 완료 출력을 구분할 수 있습니다

## Tailnet 접근 (권장)

### 통합 Tailscale Serve (선호)

Gateway를 루프백에 유지하고 Tailscale Serve가 HTTPS로 프록시하게 합니다:

```bash
openclaw gateway --tailscale serve
```

열기:

- `https://<magicdns>/` (또는 구성된 `gateway.controlUi.basePath`)

기본적으로, `gateway.auth.allowTailscale`이 `true`인 경우 제어 UI/WebSocket Serve 요청은
Tailscale 신원 헤더 (`tailscale-user-login`)를 통해 인증할 수 있습니다. OpenClaw는
`x-forwarded-for` 주소를 `tailscale whois`로 해석하여 헤더와 일치시키고,
요청이 Tailscale의 `x-forwarded-*` 헤더와 함께 루프백에 도달할 때만 수용합니다.
Serve 트래픽에 대해서도 토큰/비밀번호를 요구하려면
`gateway.auth.allowTailscale: false` (또는 `gateway.auth.mode: "password"` 강제)로 설정하세요.
토큰 없는 Serve 인증은 gateway 호스트가 신뢰됨을 가정합니다. 해당 호스트에서
신뢰할 수 없는 로컬 코드가 실행될 수 있다면, 토큰/비밀번호 인증을 요구하세요.

### Tailnet 바인드 + 토큰

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

그런 다음:

- `http://<tailscale-ip>:18789/` (또는 구성된 `gateway.controlUi.basePath`)

UI 설정에 토큰을 붙여넣으세요 (`connect.params.auth.token`으로 전송됨).

## 보안되지 않은 HTTP

일반 HTTP (`http://<lan-ip>` 또는 `http://<tailscale-ip>`)로 대시보드를 열면,
브라우저가 **보안되지 않은 컨텍스트**에서 실행되어 WebCrypto를 차단합니다. 기본적으로,
OpenClaw는 기기 신원 없이 제어 UI 연결을 **차단**합니다.

**권장 수정:** HTTPS (Tailscale Serve) 사용 또는 UI를 로컬에서 열기:

- `https://<magicdns>/` (Serve)
- `http://127.0.0.1:18789/` (gateway 호스트에서)

**보안되지 않은 인증 토글 동작:**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth`는 제어 UI 기기 신원 또는 페어링 검사를 우회하지 않습니다.

**비상용 전용:**

```json5
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth`는 제어 UI 기기 신원 검사를 비활성화하며
심각한 보안 약화입니다. 긴급 사용 후 빠르게 복원하세요.

[Tailscale](/gateway/tailscale)에서 HTTPS 설정 안내를 참조하세요.

## UI 빌드

Gateway는 `dist/control-ui`에서 정적 파일을 제공합니다. 다음으로 빌드하세요:

```bash
pnpm ui:build # 첫 실행 시 UI 의존성 자동 설치
```

선택적 절대 경로 (고정된 에셋 URL을 원할 때):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

로컬 개발 (별도 개발 서버):

```bash
pnpm ui:dev # 첫 실행 시 UI 의존성 자동 설치
```

그런 다음 UI를 Gateway WS URL (예: `ws://127.0.0.1:18789`)로 지정하세요.

## 디버깅/테스트: 개발 서버 + 원격 Gateway

제어 UI는 정적 파일입니다; WebSocket 대상은 구성 가능하며 HTTP 원본과
다를 수 있습니다. 이는 Vite 개발 서버를 로컬에서 실행하지만 Gateway가 다른 곳에서
실행될 때 유용합니다.

1. UI 개발 서버 시작: `pnpm ui:dev`
2. 다음과 같은 URL 열기:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

선택적 일회성 인증 (필요시):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

참고:

- `gatewayUrl`은 로드 후 localStorage에 저장되고 URL에서 제거됩니다.
- `token`은 localStorage에 저장됩니다; `password`는 메모리에만 유지됩니다.
- `gatewayUrl`이 설정되면 UI는 설정이나 환경 자격 증명으로 폴백하지 않습니다.
  `token` (또는 `password`)을 명시적으로 제공하세요. 명시적 자격 증명이 없으면 오류입니다.
- Gateway가 TLS 뒤에 있을 때 (Tailscale Serve, HTTPS 프록시 등) `wss://`를 사용하세요.
- `gatewayUrl`은 클릭재킹을 방지하기 위해 최상위 창에서만 수용됩니다 (임베디드가 아님).
- 비루프백 제어 UI 배포는 `gateway.controlUi.allowedOrigins`를 명시적으로 설정해야 합니다
  (전체 오리진). 여기에는 원격 개발 설정도 포함됩니다.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`는
  Host-header 오리진 폴백 모드를 활성화하지만, 위험한 보안 모드입니다.

예시:

```json5
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

원격 접근 설정 세부사항: [원격 접근](/gateway/remote).
