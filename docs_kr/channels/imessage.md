---
summary: "레거시 imsg 방식을 통한 iMessage 연동 가이드. 신규 사용자는 블루버블(BlueBubbles)을 권장합니다."
read_when:
  - 기존에 imsg를 사용하여 아이메시지를 연동 중일 때
  - 블루버블을 사용할 수 없는 환경에서 대안을 찾을 때
title: "iMessage (레거시)"
---

# iMessage (레거시: imsg)

<Warning>
새로 iMessage를 연동하시려는 분들은 <a href="/channels/bluebubbles">블루버블(BlueBubbles)</a> 채널 사용을 강력히 권장합니다. 이 `imsg` 방식은 레거시이며 향후 지원이 중단될 수 있습니다.
</Warning>

이 방식은 macOS의 `imsg` CLI 도구를 사용하여 아이메시지를 직접 제어합니다.

## 설정 전 확인 사항
- **macOS 전용**: 반드시 메시지 앱이 로그인된 Mac에서 실행해야 합니다.
- **권한 승인**: 메시지 데이터베이스 접근을 위해 '전체 디스크 접근 권한(Full Disk Access)'이 필요하며, 메시지 전송을 위한 '자동화(Automation)' 권한 승인이 필요합니다.

## 빠른 설정 방법
1.  Mac에서 `imsg`를 설치합니다: `brew install steipete/tap/imsg`
2.  `openclaw.json`에 경로를 설정합니다:
    ```json5
    {
      "channels": {
        "imessage": {
          "enabled": true,
          "cliPath": "/usr/local/bin/imsg",
          "dbPath": "/Users/사용자이름/Library/Messages/chat.db"
        }
      }
    }
    ```
3.  게이트웨이를 실행하고 페어링 코드로 승인합니다.

## 한계점
- 원격 서버(Linux 등)에서 실행하려면 SSH를 통한 복잡한 설정이 필요합니다.
- 메시지 수정, 전송 취소 등 최신 기능을 지원하지 않습니다. (이러한 기능은 블루버블에서 지원합니다.)

---
권장되는 연동 방법은 [블루버블 가이드](/channels/bluebubbles)를 참고하세요.
---
