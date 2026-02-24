---
summary: "게이트웨이 대시보드 및 WebSocket을 위한 Tailscale Serve/Funnel 통합 가이드"
read_when:
  - 게이트웨이 관리 UI를 로컬 호스트 외부로 노출하고 싶을 때
  - Tailnet이나 공용 인터넷을 통한 대시보드 접근을 자동화하고 싶을 때
title: "Tailscale 연동"
---

# Tailscale 연동 (게이트웨이 대시보드)

OpenClaw는 게이트웨이 대시보드와 WebSocket 포트를 내부 네트워크(Tailnet)나 공용 인터넷에 자동으로 노출하기 위해 **Tailscale Serve** 및 **Funnel** 기능을 지원합니다. 이를 통해 게이트웨이는 보안을 위해 루프백(`127.0.0.1`)에만 유지하면서도, Tailscale이 제공하는 HTTPS와 라우팅 기능을 활용할 수 있습니다.

## 주요 모드

- **`serve`**: Tailnet(내부 네트워크) 사용자에게만 대시보드를 노출합니다. 별도의 토큰 입력 없이 Tailscale 사용자 인증 정보를 활용할 수 있습니다.
- **`funnel`**: 공용 인터넷(HTTPS)을 통해 대시보드를 노출합니다. 보안을 위해 공동 비밀번호(`password`) 설정이 필수입니다.
- **`off`**: Tailscale 자동화 기능을 사용하지 않습니다 (기본값).

## 인증 방식

Tailscale `serve` 모드를 사용하고 `gateway.auth.allowTailscale` 옵션이 켜져 있다면, OpenClaw는 Tailscale의 사용자 정보를 확인하여 **별도의 토큰이나 비밀번호 없이** 접속을 허용할 수 있습니다. (단, API 엔드포인트는 여전히 토큰/비밀번호 인증이 필요합니다.)

## 설정 예시 (`openclaw.json`)

### 1. Tailnet 전용 (Serve)
내부 네트워크에서만 안전하게 접속하고 싶을 때 사용합니다.
```json5
{
  "gateway": {
    "bind": "loopback",
    "tailscale": { "mode": "serve" }
  }
}
```
접속 주소: `https://<MagicDNS_이름>/`

### 2. 공용 인터넷 노출 (Funnel)
외부에서 비밀번호를 통해 접속하고 싶을 때 사용합니다.
```json5
{
  "gateway": {
    "bind": "loopback",
    "tailscale": { "mode": "funnel" },
    "auth": { "mode": "password", "password": "비밀번호_입력" }
  }
}
```

## 주요 참고 사항

- **준비 사항**: `tailscale` CLI가 설치되어 있고 로그인된 상태여야 합니다.
- **보안**: `funnel` 모드는 공용 인터넷에 노출되므로 반드시 강력한 비밀번호를 설정하십시오.
- **노드 연결**: iOS/Android 앱 등 노드 클라이언트도 Tailscale을 통해 게이트웨이에 접속할 수 있습니다.

상세한 CLI 옵션이나 기술적 제한 사항은 [영문 원본 문서](/gateway/tailscale)를 참고하십시오.
---
