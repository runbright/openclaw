---
summary: "페어링(Pairing) 개요: 나에게 메시지를 보낼 수 있는 사람과 게이트웨이에 참여할 노드 승인하기"
read_when:
  - DM 접근 제어를 설정할 때
  - 새로운 iOS/안드로이드 노드를 연결할 때
  - OpenClaw의 보안 상태를 점검할 때
title: "페어링"
---

# 페어링 (Pairing)

"페어링"은 OpenClaw의 보안을 지키는 가장 중요한 **소유자 승인** 단계입니다. 다음 두 가지 경우에 사용됩니다:

1.  **DM 페어링**: 누가 봇에게 메시지를 보낼 수 있는지 결정합니다.
2.  **노드 기기 페어링**: 어떤 스마트폰이나 컴퓨터가 게이트웨이 네트워크에 참여할지 결정합니다.

---

## 1) DM 페어링 (채팅 접근 권한)

채널 설정에서 `dmPolicy`가 `pairing`으로 되어 있으면, 처음 메시지를 보낸 사람에게 8자리의 대문자 코드가 전송됩니다. 해당 메시지는 사용자가 승인하기 전까지는 에이전트에게 전달되지 않습니다.

- **유효 시간**: 코드는 생성 후 **1시간** 동안만 유효합니다.
- **승인 방법 (CLI)**:
  ```bash
  openclaw pairing list telegram    # 대기 목록 확인
  openclaw pairing approve telegram <코드> # 승인 완료
  ```

## 2) 노드 기기 페어링 (iOS/Android/macOS)

OpenClaw 앱(iOS/Android)을 게이트웨이에 연결할 때도 보안 승인이 필요합니다.

### 텔레그램을 통한 페어링 (권장)
1.  텔레그램에서 내 봇에게 `/pair` 명령어를 보냅니다.
2.  봇이 보내준 **설정 코드(Setup Code)**를 복사합니다.
3.  스마트폰 앱의 **Settings -> Gateway** 메뉴에서 해당 코드를 붙여넣고 연결합니다.
4.  텔레그램으로 돌아와 `/pair approve`를 입력하여 최종 승인합니다.

### CLI를 통한 기기 승인
```bash
openclaw devices list        # 연결 요청 중인 기기 확인
openclaw devices approve <요청ID> # 기기 승인
```

---

## 페어링 정보 저장 위치
이 정보들은 매우 민감한 데이터이며 아래 경로에 저장됩니다:
- **메시지 허용 목록**: `~/.openclaw/credentials/`
- **페어링된 기기 목록**: `~/.openclaw/devices/`

---
더 자세한 보안 정책은 [보안 가이드](/gateway/security)를 참고하세요.
---
