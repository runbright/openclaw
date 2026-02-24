---
summary: "SSH 터널(게이트웨이 WS) 및 Tailnet을 이용한 원격 접속 가이드"
read_when:
  - 원격 게이트웨이 설정을 실행하거나 수정할 때
title: "원격 접속 (Remote Access)"
---

# 원격 접속 (SSH, 터널 및 Tailnet)

OpenClaw는 전용 호스트(데스크톱 또는 서버)에서 하나의 게이트웨이(마스터)를 상시 실행하고, 다양한 클라이언트(CLI, macOS 앱, 모바일 노드 등)를 연결하는 "원격 접속" 환경을 지원합니다.

- **운영자 (사용자 / macOS 앱)**: SSH 터널링이 가장 범용적인 연결 방식입니다.
- **노드 (iOS/Android 기기)**: 게이트웨이 **WebSocket**에 접속합니다 (LAN, Tailnet 또는 필요에 따라 SSH 터널 사용).

## 핵심 원리

- 게이트웨이 WebSocket은 설정된 포트(기본값 18789)의 **루프백(loopback)** 인터페이스에 바인딩됩니다.
- 원격 접속을 위해서는 SSH를 통해 해당 루프백 포트를 포워딩하거나, Tailnet/VPN을 활용하여 안전한 터널을 구축해야 합니다.

## 일반적인 VPN/Tailnet 설정 사례

**게이트웨이 호스트**는 "에이전트가 거주하는 곳"입니다. 이곳에서 세션, 인증 프로파일, 채널 및 상태 정보가 관리됩니다. 여러분의 노트북이나 모바일 기기(노드)는 이 호스트에 접속하게 됩니다.

### 1) Tailnet 내에 상시 가동되는 게이트웨이 (VPS 또는 홈 서버)

전용 호스트에서 게이트웨이를 실행하고 **Tailscale** 혹은 SSH로 접속합니다.

- **가장 권장되는 방식**: `gateway.bind: "loopback"` 설정을 유지하고, 제어 UI 접속을 위해 **Tailscale Serve**를 활용합니다.
- **차선책**: 루프백 설정을 유지하되 접속이 필요한 머신에서 SSH 터널을 생성합니다.
- **사례**: [exe.dev](/install/exe-dev) (간편 VM) 또는 [Hetzner](/install/hetzner) (서버용 VPS).

노트북을 자주 끄거나 절전 모드로 두지만 에이전트는 항상 켜져 있길 원할 때 가장 좋은 모델입니다.

### 2) 집의 데스크톱이 게이트웨이, 노트북은 리모컨 역할

노트북은 에이전트를 직접 실행하지 않고 원격으로만 제어합니다:

- macOS 앱의 **Remote over SSH** 모드를 사용합니다 (Settings → General → “OpenClaw runs”).
- 앱이 자동으로 터널을 생성하고 관리하므로 웹채팅 및 헬스 체크 기능을 즉시 사용할 수 있습니다.

참고 문서: [macOS 원격 접속](/platforms/mac/remote).

### 3) 노트북이 게이트웨이, 다른 기기에서 원격 접속

노트북에서 게이트웨이를 실행하면서 외부에서 안전하게 접근합니다:

- 다른 기기에서 노트북으로 SSH 터널을 생성하거나,
- Tailscale Serve를 통해 제어 UI를 노출하고 게이트웨이 자체는 루프백 전용으로 둡니다.

참고 가이드: [Tailscale](/gateway/tailscale) 및 [웹 개요](/web).

## 명령 흐름 (무엇이 어디서 실행되는가)

하나의 게이트웨이 서비스가 상태와 채널을 소유하며, 노드들은 주변 기기(peripherals)가 됩니다.

흐름 예시 (Telegram → 노드):

- Telegram 메시지가 **게이트웨이**에 도착합니다.
- 게이트웨이가 **에이전트**를 실행하고 노드 도구(node tool)를 호출할지 결정합니다.
- 게이트웨이가 게이트웨이 WebSocket(`node.*` RPC)을 통해 **노드**를 호출합니다.
- 노드가 결과를 반환하면 게이트웨이가 다시 Telegram으로 응답을 보냅니다.

참고:
- **노드는 게이트웨이 서비스를 직접 실행하지 않습니다.** 하나의 호스트에는 하나의 게이트웨이만 실행하는 것이 원칙입니다 (격리된 프로필을 쓰는 경우는 예외. [다중 게이트웨이](/gateway/multiple-gateways) 참고).
- macOS 앱의 "노드 모드"는 게이트웨이 WebSocket에 접속된 하나의 노드 클라이언트일 뿐입니다.

## SSH 터널 생성 (CLI 및 도구)

원격 게이트웨이 WebSocket에 연결하기 위한 로컬 터널을 생성합니다:

```bash
ssh -N -L 18789:127.0.0.1:18789 사용자@호스트
```

터널이 연결된 상태에서:
- `openclaw health` 및 `openclaw status --deep` 명령은 `ws://127.0.0.1:18789`를 통해 원격 게이트웨이에 도달합니다.
- `openclaw gateway {status,health,send,agent,call}` 명령 실행 시 필요에 따라 `--url` 옵션으로 포워딩된 URL을 지정할 수 있습니다.

참고: `18789`를 본인이 설정한 `gateway.port`로 변경하세요.
참고: `--url` 옵션을 사용할 경우 CLI는 설정 파일이나 환경 변수의 인증 정보를 자동으로 사용하지 않습니다. 반드시 `--token` 또는 `--password`를 명시적으로 포함해야 합니다.

## CLI 원격 기본값 설정

CLI 명령이 기본적으로 원격 대상을 바라보도록 설정 파일에 기록할 수 있습니다:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

게이트웨이가 루프백 전용으로 설정된 경우 URL을 `ws://127.0.0.1:18789`로 유지하고 먼저 SSH 터널을 열어두어야 합니다.

## 인증 정보 우선순위

게이트웨이 호출 및 프로브 시의 인증 정보 결정 규칙은 다음과 같습니다:

1. 명시적 인자(`--token`, `--password` 등)가 가장 높은 우선순위를 갖습니다.
2. 로컬 모드 기본값: `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` 순으로 확인합니다.
3. 원격 모드 기본값: `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` 순으로 확인합니다.

## SSH를 통한 채팅 UI 이용

WebChat은 더 이상 별도의 HTTP 포트를 사용하지 않으며, 게이트웨이 WebSocket에 직접 접속합니다.

- SSH를 통해 `18789` 포트를 포워딩한 뒤, 클라이언트를 `ws://127.0.0.1:18789`로 연결하세요.
- macOS의 경우 터널을 자동으로 관리해 주는 앱의 "Remote over SSH" 모드를 사용하는 것이 편리합니다.

## 보안 수칙 (원격 및 VPN 사용 시)

요약: **반드시 필요한 경우가 아니라면 게이트웨이를 루프백 전용으로 유지하세요.**

- **루프백 + SSH/Tailscale Serve** 방식이 가장 안전한 기본값입니다 (외부 노출 없음).
- 루프백이 아닌 **외부 바인딩**(`lan`, `tailnet`, `custom` 등) 시에는 반드시 인증 토큰이나 비밀번호를 사용해야 합니다.
- `gateway.remote.token`은 원격 CLI 호출을 위한 설정일 뿐 로컬 인증을 활성화하는 설정이 아닙니다.
- **Tailscale Serve**는 `gateway.auth.allowTailscale: true` 설정 시 고유 헤더를 통해 제어 UI/WebSocket 트래픽을 인증할 수 있습니다. 단, HTTP API 엔드포인트는 여전히 토큰/비번 인증이 필요합니다.

상세 내용: [보안](/gateway/security).
