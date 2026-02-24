---
summary: "Gateway 대시보드 (제어 UI) 접근 및 인증"
read_when:
  - 대시보드 인증 또는 노출 모드 변경 시
title: "대시보드"
---

# 대시보드 (제어 UI)

Gateway 대시보드는 기본적으로 `/`에서 제공되는 브라우저 제어 UI입니다
(`gateway.controlUi.basePath`로 재정의 가능).

빠른 열기 (로컬 Gateway):

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (또는 [http://localhost:18789/](http://localhost:18789/))

주요 참고자료:

- [제어 UI](/web/control-ui) 사용법 및 UI 기능.
- [Tailscale](/gateway/tailscale) Serve/Funnel 자동화.
- [웹 인터페이스](/web) 바인드 모드 및 보안 참고사항.

인증은 WebSocket 핸드셰이크에서 `connect.params.auth`
(토큰 또는 비밀번호)를 통해 시행됩니다. [Gateway 설정](/gateway/configuration)의 `gateway.auth`를 참조하세요.

보안 참고: 제어 UI는 **관리자 인터페이스** (채팅, 설정, exec 승인)입니다.
공개적으로 노출하지 마세요. UI는 첫 로드 후 `localStorage`에 토큰을 저장합니다.
localhost, Tailscale Serve 또는 SSH 터널을 선호하세요.

## 빠른 경로 (권장)

- 온보딩 후 CLI가 대시보드를 자동으로 열고 깔끔한 (토큰 없는) 링크를 출력합니다.
- 언제든 다시 열기: `openclaw dashboard` (링크 복사, 가능하면 브라우저 열기, 헤드리스이면 SSH 힌트 표시).
- UI가 인증을 요청하면, `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`)의 토큰을 제어 UI 설정에 붙여넣으세요.

## 토큰 기본 (로컬 vs 원격)

- **Localhost**: `http://127.0.0.1:18789/`를 여세요.
- **토큰 출처**: `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`); 연결 후 UI가 localStorage에 사본을 저장합니다.
- **Localhost가 아닌 경우**: Tailscale Serve (제어 UI/WebSocket에 대해 `gateway.auth.allowTailscale: true`이면 토큰 없이 가능, 신뢰할 수 있는 gateway 호스트를 가정; HTTP API는 여전히 토큰/비밀번호 필요), 토큰이 있는 tailnet 바인드, 또는 SSH 터널을 사용하세요. [웹 인터페이스](/web)를 참조하세요.

## "unauthorized" / 1008이 보이는 경우

- gateway에 접근 가능한지 확인 (로컬: `openclaw status`; 원격: SSH 터널 `ssh -N -L 18789:127.0.0.1:18789 user@host` 후 `http://127.0.0.1:18789/` 열기).
- gateway 호스트에서 토큰 조회: `openclaw config get gateway.auth.token` (또는 생성: `openclaw doctor --generate-gateway-token`).
- 대시보드 설정에서 인증 필드에 토큰을 붙여넣은 다음 연결하세요.
