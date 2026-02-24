---
summary: "Bonjour/mDNS 탐색 기술과 디버깅 가이드 (게이트웨이 비컨, 클라이언트 및 주요 실패 사례)"
read_when:
  - macOS/iOS에서 Bonjour 탐색 관련 문제를 디버그할 때
  - mDNS 서비스 유형, TXT 레코드 또는 탐색 UX를 변경할 때
title: "Bonjour 탐색"
---

# Bonjour / mDNS 탐색

OpenClaw는 Bonjour(mDNS / DNS-SD)를 사용하여 같은 로컬 네트워크(LAN)에 있는 활성화된 게이트웨이(WebSocket 엔드포인트)를 자동으로 찾아냅니다. 이는 사용자 편의를 위한 기능이며, SSH나 Tailscale 기반의 연결을 대체하는 것은 아닙니다.

## 주요 연결 방식

### 1. 로컬 네트워크 (LAN)
동일한 Wi-Fi나 이더넷에 연결된 경우 mDNS(Multicast DNS)를 통해 자동으로 게이트웨이를 탐색합니다.

### 2. Tailscale 환경 (Wide-Area Bonjour)
서로 다른 네트워크에 있더라도 Tailscale을 사용한다면 **유니캐스트 DNS-SD**를 통해 동일한 탐색 경험을 유지할 수 있습니다.

- **게이트웨이 설정**:
  ```json5
  {
    "gateway": { "bind": "tailnet" },
    "discovery": { "wideArea": { "enabled": true } }
  }
  ```
- **DNS 서버 구성**: `openclaw dns setup --apply` 명령어를 통해 게이트웨이 기기에 CoreDNS를 설치하고 Tailscale 전용 도메인(예: `openclaw.internal.`)을 구성할 수 있습니다.

## 서비스 상세 정보 (TXT 레코드)

게이트웨이는 탐색을 돕기 위해 다음과 같은 메타데이터(힌트)를 함께 배포합니다:
- `displayName`: 게이트웨이의 별칭
- `lanHost`: 호스트 이름 (예: `macbook.local`)
- `gatewayPort`: WebSocket 및 HTTP 포트 (기본 18789)
- `sshPort`: SSH 포트 (기본 22)
- `gatewayTls`: TLS 활성화 여부

> **보안 주의**: Bonjour TXT 레코드는 인증되지 않은 정보입니다. 클라이언트는 이를 절대적인 정보로 믿지 말고, 연결 시 반드시 인증 과정을 거쳐야 합니다. 특히 iOS/Android 노드는 탐색된 게이트웨이에 첫 연결 시 사용자 확인을 요구합니다.

## 디버깅 방법 (macOS 전용)

- **인스턴스 목록 확인**: 
  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```
- **특정 인스턴스 상세 정보 조회**:
  ```bash
  dns-sd -L "<인스턴스이름>" _openclaw-gw._tcp local.
  ```

## 주요 실패 사례 및 해결책

- **탐색은 되는데 연결이 안 되는 경우**: 호스트 이름에 이모지나 복잡한 문장 부호가 포함되어 있는지 확인하세요. 단순한 영문 이름으로 변경 후 게이트웨이를 재시작하는 것이 좋습니다.
- **네트워크 간 탐색 불가**: 멀티캐스트 mDNS는 네트워크 경계를 넘지 못합니다. Tailscale이나 SSH 터널링을 사용하십시오.
- **Wi-Fi 제한**: 일부 공공/회사 Wi-Fi는 보안 정책상 mDNS 기능을 차단합니다.

## 설정 제어

- **탐색 비활성화**: 환경 변수 `OPENCLAW_DISABLE_BONJOY=1`을 설정하거나 게이트웨이 설정에서 탐색 관련 옵션을 끕니다.
- **SSH 포트 수동 지정**: `OPENCLAW_SSH_PORT` 환경 변수를 통해 광고할 SSH 포트를 변경할 수 있습니다.

상세 기술 사양이나 페어링 절차는 [탐색 정책](/gateway/discovery) 및 [게이트웨이 페어링](/gateway/pairing) 문서를 참고하십시오.
---
