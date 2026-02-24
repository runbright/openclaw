---
summary: "NIP-04 암호화 메시지를 통한 Nostr DM 채널 설정 가이드"
read_when:
  - Nostr를 통해 OpenClaw DM을 받고 싶을 때
  - 탈중앙화 메시징 환경을 구축할 때
title: "Nostr"
---

# Nostr (탈중앙화 메시징)

Nostr은 탈중앙화된 소셜 네트워크 프로토콜입니다. OpenClaw는 이 채널을 통해 NIP-04 표준에 따른 암호화된 개인 메시지(DM)를 수신하고 응답할 수 있습니다.

## 플러그인 설치

Nostr은 선택적 플러그인으로 제공됩니다:

```bash
openclaw plugins install @openclaw/nostr
```

## 빠른 설정 가이드

1. **키 쌍(Keypair) 생성**: 에이전트 전용 개인키(`nsec`)와 공개키(`npub`)가 필요합니다.
2. **OpenClaw 설정 (`openclaw.json`)**:
```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://nos.lol"]
    }
  }
}
```
3. **환경 변수 설정**: 개인키를 노출하지 않도록 환경 변수로 관리하는 것을 권장합니다.
```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```
4. **게이트웨이 실행**: 설정을 마친 후 게이트웨이를 시작합니다.

## 접근 제어 (Access Control)

- **pairing** (기본값): 모르는 사용자로부터 DM이 오면 페어링 코드를 발송합니다.
- **allowlist**: `allowFrom` 목록에 명시된 공개키(`npub` 또는 hex) 사용자만 허용합니다.
- **open**: 모든 사용자의 DM을 허용합니다 (`allowFrom: ["*"]` 필요).

## 릴레이 (Relays)

Nost는 메시지 전달을 위해 릴레이 서버를 사용합니다. 안정성을 위해 2~3개의 릴레이를 사용하는 것이 좋으며, `wss://relay.damus.io` 등이 기본으로 설정되어 있습니다.

## 지원 프로토콜

| NIP | 상태 | 설명 |
| :--- | :--- | :--- |
| NIP-01 | ✅ 지원 | 기본 이벤트 형식 및 프로필 메타데이터 |
| NIP-04 | ✅ 지원 | 암호화된 DM (`kind:4`) |
| NIP-17 | 🚧 예정 | 선물 포장(Gift-wrapped) DM |

## 주의사항 및 제한 사항 (현재 버전)

- **개인 메시지만 지원**: 현재는 그룹 채팅 기능이 지원되지 않습니다.
- **미디어 누락**: 첨부 파일이나 이미지 전송 기능은 아직 제공되지 않습니다.
- **보안**: 개인키(`nsec`)가 유출되지 않도록 각별히 주의하십시오. 환경 변수 사용을 강력히 권장합니다.

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
