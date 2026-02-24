---
summary: "Tlon/Urbit 연동 상태, 지원 기능 및 에이전트 설정 가이드"
read_when:
  - Tlon/Urbit 채널 기능을 구현하거나 디버깅할 때
title: "Tlon"
---

# Tlon (플러그인)

Tlon은 Urbit(어빗) 위에 구축된 탈중앙화 메신저입니다. OpenClaw는 사용자의 Urbit 쉽(Ship)에 연결하여 개인 메시지(DM)와 그룹 채팅 메시지에 응답할 수 있습니다.

## 플러그인 설치

Tlon은 전용 플러그인 설치가 필요합니다:

```bash
openclaw plugins install @openclaw/tlon
```

## 설정 가이드

1. **쉽(Ship) 정보 확인**: 본인의 쉽 이름(예: `~sampel-palnet`), 접속 URL, 그리고 로그인 코드가 필요합니다.
2. **OpenClaw 설정 (`openclaw.json`)**:

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://쉽_접속_주소",
      code: "로그인_코드",
    },
  },
}
```

3. **기타 네트워크 설정**: 만약 쉽이 로컬 네트워크(LAN)에 있다면 `allowPrivateNetwork: true` 설정을 추가해야 연동이 가능합니다.

## 접근 제어 (Access Control)

- **DM 허용 목록**: `dmAllowlist`에 특정 쉽 이름을 나열하여 대화 상대를 제한할 수 있습니다. 비워두면 모든 사용자의 DM을 허용합니다.
- **그룹 권한**: 그룹 채팅에서는 기본적으로 @멘션이 필요합니다. `authorization` 설정을 통해 채널별로 응답 규칙(`mode: "restricted"` 또는 `mode: "open"`)을 다르게 지정할 수 있습니다.

## 주요 기능 및 특징

- **자동 채널 발견**: 봇이 참여 중인 채널을 자동으로 감지합니다 (`autoDiscoverChannels`).
- **스레드 응답**: 들어온 메시지가 스레드라면 에이전트도 해당 스레드 내에서 답변합니다.
- **미디어 제한**: 현재 이미지나 파일의 직접 업로드는 지원하지 않으며, 텍스트 메시지에 URL을 추가하는 방식으로 대체됩니다.
- **리액션 및 투표**: 아직 지원되지 않습니다.

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
