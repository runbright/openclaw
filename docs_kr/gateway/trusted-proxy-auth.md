---
summary: "신뢰할 수 있는 역방향 프록시(Pomerium, Caddy, nginx + OAuth 등)에 게이트웨이 인증을 위임하는 방법 안내"
read_when:
  - ID 인식 프록시(identity-aware proxy) 뒤에서 OpenClaw를 실행할 때
  - OpenClaw 앞단에 OAuth 장착 프록시를 설정할 때
  - 역방향 프록시 환경에서 WebSocket 1008 인증 오류를 해결할 때
title: "프록시 인증 (Trusted Proxy Auth)"
---

# 프록시 인증 (Trusted Proxy Auth)

> ⚠️ **보안 주의 사항.** 이 모드는 인증 처리를 전적으로 역방향 프록시(Reverse Proxy)에 위임합니다. 설정이 잘못될 경우 권한 없는 사용자가 게이트웨이에 접근할 수 있습니다. 활성화하기 전에 이 문서를 주의 깊게 읽어주세요.

## 사용 목적

다음과 같은 상황에서 `trusted-proxy` 인증 모드를 사용합니다:
- **ID 인식 프록시**(Pomerium, Caddy + OAuth, nginx + oauth2-proxy 등) 뒤에서 OpenClaw를 실행할 때
- 프록시가 모든 인증을 처리하고 사용자 식별 정보를 헤더로 전달할 때
- 브라우저가 WebSocket 요청 시 페이로드에 토큰을 담지 못해 발생하는 `1008 unauthorized` 오류를 해결할 때

## 작동 원리

1. 역방향 프록시가 사용자 인증(OAuth, OIDC 등)을 수행합니다.
2. 프록시는 인증된 사용자의 식별 정보(예: `nick@example.com`)를 헤더(`x-forwarded-user`)에 담아 OpenClaw로 전달합니다.
3. OpenClaw는 요청이 **신뢰할 수 있는 프록시 IP**(`gateway.trustedProxies`)로부터 왔는지 확인합니다.
4. OpenClaw는 지정된 헤더에서 사용자 식별 정보를 추출하고 권한을 부여합니다.

## 구성 설정 예시

```json5
{
  gateway: {
    // 프록시와 같은 호스트라면 loopback, 원격이라면 해당 IP 지정
    bind: "loopback",

    // 중요: 프록시 서버의 IP 주소만 여기에 추가하세요
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 사용자 식별 정보가 담긴 헤더 이름 (필수)
        userHeader: "x-forwarded-user",

        // 선택: 검증을 위해 반드시 포함되어야 하는 추가 헤더들
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // 선택: 특정 사용자만 허용 (비어있으면 인증된 모든 사용자 허용)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

## 프록시 설정 사례

### Caddy + OAuth 사용 시
Caddyfile 설정 예시:
```caddy
openclaw.example.com {
    authenticate with oauth2_provider
    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

OpenClaw 설정 예시:
```json5
{
  gateway: {
    trustedProxies: ["127.0.0.1"],
    auth: {
      mode: "trusted-proxy",
      trustedProxy: { userHeader: "x-forwarded-user" }
    }
  }
}
```

## 보안 체크리스트

프록시 인증을 활성화하기 전에 다음을 확인하세요:
- [ ] **프록시가 유일한 경로인가**: 게이트웨이 포트가 프록시 IP 이외의 모든 접근으로부터 방화벽으로 차단되어 있는가?
- [ ] **IP 목록 최소화**: `trustedProxies`에 서브넷 전체가 아닌 실제 프록시 IP만 지정되어 있는가?
- [ ] **헤더 변조 방지**: 프록시가 클라이언트로부터 오는 `x-forwarded-*` 헤더를 덮어쓰거나 무시하도록 설정되어 있는가?
- [ ] **허용 목록 설정**: `allowUsers`를 사용하여 신뢰할 수 있는 사용자만 접근하도록 제한하는 것을 권장합니다.

## 트러블슈팅

- **"trusted_proxy_untrusted_source"**: 요청이 `gateway.trustedProxies`에 등록되지 않은 IP에서 왔습니다. 프록시 서버의 실제 내부 IP를 확인하세요.
- **"trusted_proxy_user_missing"**: 프록시가 사용자 식별 헤더를 보내지 않았습니다. 프록시 설정에서 헤더 전달 옵션을 확인하세요.
- **WebSocket 연결 실패**: 프록시가 WebSocket 업그레이드 요청 시에도 동일한 인증 헤더를 전달하는지 확인하세요.
---
