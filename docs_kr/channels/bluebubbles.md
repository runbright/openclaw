---
summary: "블루버블(BlueBubbles)을 통한 iMessage 연동 및 설정 가이드. iMessage 통합 시 가장 권장되는 방법입니다."
read_when:
  - 아이메시지(iMessage)를 OpenClaw에 연결하고 싶을 때
  - 블루버블 서버 설정 및 웹훅 연결 방법을 알고 싶을 때
title: "블루버블 (iMessage)"
---

# 블루버블 (BlueBubbles - iMessage 연동)

블루버블 채널은 macOS 기반의 블루버블 서버를 통해 iMessage를 활용할 수 있게 해줍니다. 풍부한 API 기능을 제공하므로 **iMessage 연동을 위한 가장 권장되는 방식**입니다.

## 주요 기능
- **메시지 관리**: 메시지 전송, 읽음 처리, 타이핑 표시(입력 중...)를 지원합니다.
- **고급 기능**: 메시지 수정(`Edit`), 전송 취소(`Unsend`), 메시지에 리액션 남기기(`Tapback`) 등이 가능합니다.
- **그룹 대화**: 그룹 대화 참여, 그룹 이름 변경, 멤버 추가/제거 등을 관리할 수 있습니다.
- **미디어**: 이미지, 파일 전송 및 음성 메시지 전송을 지원합니다.

## 빠른 설정 방법

1.  **서버 설치**: macOS 기기에 [BlueBubbles Server](https://bluebubbles.app/)를 설치하고 설정합니다.
2.  **API 비밀번호 설정**: 블루버블 설정에서 Web API를 활성화하고 비밀번호를 정합니다.
3.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "channels": {
        "bluebubbles": {
          "enabled": true,
          "serverUrl": "http://맥-아이피:1234",
          "password": "서버-비밀번호",
          "webhookPath": "/bluebubbles-webhook"
        }
      }
    }
    ```
4.  **웹훅 등록**: 블루버블 서버 설정의 웹훅(Webhooks) 섹션에 OpenClaw 게이트웨이 주소를 입력합니다. (예: `http://서버-아이피:18789/bluebubbles-webhook?password=비밀번호`)

## 보안 및 페어링
- **DM 정책**: 기본적으로 `pairing` 모드가 적용됩니다. 처음 메시지를 보낼 때 페어링 코드로 승인하세요.
- **허용 목록**: `allowFrom` 설정을 통해 특정 번호나 이메일 주소만 대화를 허용할 수 있습니다.

## 꿀팁: 메시지 앱 항상 켜두기
일부 macOS 설정에서는 메시지 앱이 배경으로 넘어가면 알림이 오지 않을 수 있습니다. 5분마다 메시지 앱을 살짝 건드려주는 애플스크립트(AppleScript)를 LaunchAgent로 등록해두면 안정적으로 사용할 수 있습니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
