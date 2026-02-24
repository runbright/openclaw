---
summary: "Gateway 웹 인터페이스: 제어 UI, 바인드 모드 및 보안"
read_when:
  - Tailscale을 통해 Gateway에 접근하려는 경우
  - 브라우저 제어 UI 및 설정 편집을 원하는 경우
title: "웹"
---

# 웹 (Gateway)

Gateway는 Gateway WebSocket과 동일한 포트에서 작은 **브라우저 제어 UI** (Vite + Lit)를 제공합니다:

- 기본값: `http://<host>:18789/`
- 선택적 접두사: `gateway.controlUi.basePath` 설정 (예: `/openclaw`)

기능은 [제어 UI](/web/control-ui)에 있습니다.
이 페이지는 바인드 모드, 보안, 웹 대면 인터페이스에 초점을 맞춥니다.

## 웹훅

`hooks.enabled=true`인 경우, Gateway는 동일한 HTTP 서버에서 작은 웹훅 엔드포인트도 노출합니다.
인증 + 페이로드에 대해서는 [Gateway 설정](/gateway/configuration) -> `hooks`를 참조하세요.

## 설정 (기본 켜짐)

제어 UI는 에셋이 있을 때 (`dist/control-ui`) **기본적으로 활성화**됩니다.
설정을 통해 제어할 수 있습니다:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath 선택사항
  },
}
```

## Tailscale 접근

### 통합 Serve (권장)

Gateway를 루프백에 유지하고 Tailscale Serve가 프록시하게 합니다:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

그런 다음 gateway를 시작합니다:

```bash
openclaw gateway
```

열기:

- `https://<magicdns>/` (또는 구성된 `gateway.controlUi.basePath`)

### Tailnet 바인드 + 토큰

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

그런 다음 gateway를 시작합니다 (비루프백 바인드에는 토큰 필요):

```bash
openclaw gateway
```

열기:

- `http://<tailscale-ip>:18789/` (또는 구성된 `gateway.controlUi.basePath`)

### 공개 인터넷 (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // 또는 OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## 보안 참고사항

- Gateway 인증은 기본적으로 필요합니다 (토큰/비밀번호 또는 Tailscale 신원 헤더).
- 비루프백 바인드는 여전히 공유 토큰/비밀번호를 **필요로 합니다** (`gateway.auth` 또는 env).
- 마법사가 기본적으로 gateway 토큰을 생성합니다 (루프백에서도).
- UI는 `connect.params.auth.token` 또는 `connect.params.auth.password`를 전송합니다.
- 비루프백 제어 UI 배포의 경우, `gateway.controlUi.allowedOrigins`를
  명시적으로 설정하세요 (전체 오리진). 없으면 gateway 시작이 기본적으로 거부됩니다.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`는
  Host-header 오리진 폴백 모드를 활성화하지만, 위험한 보안 약화입니다.
- Serve를 사용하면, `gateway.auth.allowTailscale`이 `true`일 때 Tailscale 신원 헤더가
  제어 UI/WebSocket 인증을 충족시킬 수 있습니다 (토큰/비밀번호 불필요).
  HTTP API 엔드포인트는 여전히 토큰/비밀번호가 필요합니다.
  명시적 자격 증명을 요구하려면 `gateway.auth.allowTailscale: false`로 설정하세요.
  [Tailscale](/gateway/tailscale) 및 [보안](/gateway/security)을 참조하세요. 이
  토큰 없는 흐름은 gateway 호스트가 신뢰됨을 가정합니다.
- `gateway.tailscale.mode: "funnel"`은 `gateway.auth.mode: "password"` (공유 비밀번호)를 필요로 합니다.

## UI 빌드

Gateway는 `dist/control-ui`에서 정적 파일을 제공합니다. 다음으로 빌드하세요:

```bash
pnpm ui:build # 첫 실행 시 UI 의존성 자동 설치
```
