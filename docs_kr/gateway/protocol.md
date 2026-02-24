---
summary: "게이트웨이 WebSocket 프로토콜: 핸드셰이크, 프레임 구조 및 버전 관리 안내"
read_when:
  - 게이트웨이 WebSocket 클라이언트를 구현하거나 업데이트할 때
  - 프로토콜 불일치나 접속 실패 문제를 디버깅할 때
  - 프로토콜 스키마 및 모델을 다시 생성할 때
title: "게이트웨이 프로토콜"
---

# 게이트웨이 프로토콜 (WebSocket)

게이트웨이 WebSocket 프로토콜은 OpenClaw의 **단일 제어 평면(Control plane) 및 노드 전송 계층**입니다. 모든 클라이언트(CLI, 웹 UI, macOS 앱, iOS/Android 노드 등)는 WebSocket을 통해 접속하며, 핸드셰이크 시점에 자신의 **역할(Role)**과 **권한 범위(Scope)**를 선언합니다.

## 전송 계층 (Transport)

- JSON 페이로드를 사용하는 WebSocket 텍스트 프레임을 이용합니다.
- 첫 번째 프레임은 반드시 `connect` 요청이어야 합니다.

## 핸드셰이크 (Handshake)

게이트웨이 → 클라이언트 (접속 전 챌린지):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

클라이언트 → 게이트웨이 (접속 요청):

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "auth": { "token": "…" },
    "device": {
      "id": "기기_핑거프린트",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

게이트웨이 → 클라이언트 (응답):

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

## 프레임 구조 (Framing)

- **요청 (Request)**: `{type:"req", id, method, params}`
- **응답 (Response)**: `{type:"res", id, ok, payload|error}`
- **이벤트 (Event)**: `{type:"event", event, payload, seq?, stateVersion?}`

## 역할(Roles) 및 권한 범위(Scopes)

### 역할 (Roles)
- `operator`: 제어 평면 클라이언트 (CLI, UI, 자동화 도구 등).
- `node`: 기능 호스트 (카메라, 화면, 시스템 실행 등 기기 기능을 제공하는 주체).

### 권한 범위 (Scopes - 운영자 기준)
- `operator.read`: 정보 읽기 권한.
- `operator.write`: 설정 및 데이터 쓰기 권한.
- `operator.admin`: 관리자 권한.
- `operator.approvals`: 실행 승인 권한.
- `operator.pairing`: 기기 페어링 관리 권한.

### 노드 기능 정의 (Caps/Commands/Permissions)
노드는 접속 시 다음과 같은 기능 선언을 포함합니다:
- `caps`: 상위 수준의 기능 카테고리.
- `commands`: 실행 가능한 명령 허용 목록.
- `permissions`: 세부적인 권한 토글 (예: `screen.record`, `camera.capture`).

## 프로토콜 특징

### 실행 승인 (Exec approvals)
실행 요청에 승인이 필요한 경우, 게이트웨이는 `exec.approval.requested` 이벤트를 브로드캐스트합니다. 운영자 클라이언트는 `exec.approval.resolve` 메서드를 호출하여 이를 승인하거나 거부합니다.

### 버전 관리 (Versioning)
- `PROTOCOL_VERSION`은 소스 코드 내에서 관리됩니다.
- 클라이언트는 지원하는 프로토콜 버전 범위(`minProtocol` ~ `maxProtocol`)를 전송하며, 서버는 일치하지 않을 경우 접속을 거부합니다.

### 인증 (Auth)
- `OPENCLAW_GATEWAY_TOKEN`이 설정된 경우 토큰이 일치해야 소켓이 유지됩니다.
- 페어링 완료 후 게이트웨이는 역할과 권한 범위에 한정된 **기기 토큰(device token)**을 발급합니다. 클라이언트는 이후 접속을 위해 이 토큰을 저장해 두어야 합니다.

### 기기 정체성 및 페어링
- 모든 노드와 운영자 클라이언트는 접속 시 고유한 기기 식별자(`device.id`)를 포함해야 합니다.
- 게이트웨이는 기기와 역할별로 토큰을 발급하며, 새로운 기기 ID에 대해서는 페어링 승인이 필요합니다.
- 모든 연결은 서버가 제공한 `connect.challenge` 챌린지 값을 서명해야 합니다.

### 보안 (TLS + Pinning)
- WebSocket 연결 시 TLS가 지원됩니다.
- 클라이언트는 선택적으로 게이트웨이 인증서의 핑거프린트를 고정(Pinning)하여 보안을 강화할 수 있습니다.
---
