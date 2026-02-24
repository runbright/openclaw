---
summary: "macOS 권한 지속성 (TCC) 및 서명 요구사항"
read_when:
  - 누락되거나 멈춘 macOS 권한 프롬프트 디버깅
  - macOS 앱 패키징 또는 서명
  - 번들 ID 또는 앱 설치 경로 변경
title: "macOS 권한"
---

# macOS 권한 (TCC)

macOS 권한 부여는 취약합니다. TCC는 권한 부여를 앱의 코드 서명, 번들 식별자 및 디스크상의 경로와 연결합니다. 이 중 하나라도 변경되면 macOS는 앱을 새 앱으로 취급하며 프롬프트를 삭제하거나 숨길 수 있습니다.

## 안정적인 권한을 위한 요구사항

- 동일한 경로: 고정된 위치에서 앱을 실행하세요 (OpenClaw의 경우 `dist/OpenClaw.app`).
- 동일한 번들 식별자: 번들 ID를 변경하면 새 권한 ID가 생성됩니다.
- 서명된 앱: 서명되지 않거나 임시 서명된 빌드는 권한을 유지하지 않습니다.
- 일관된 서명: 실제 Apple Development 또는 Developer ID 인증서를 사용하여 리빌드 간에 서명이 안정적이도록 하세요.

임시 서명은 매 빌드마다 새 ID를 생성합니다. macOS가 이전 부여를 잊어버리고, 오래된 항목이 정리될 때까지 프롬프트가 완전히 사라질 수 있습니다.

## 프롬프트가 사라졌을 때 복구 체크리스트

1. 앱을 종료하세요.
2. System Settings -> Privacy & Security에서 앱 항목을 제거하세요.
3. 동일한 경로에서 앱을 다시 실행하고 권한을 다시 부여하세요.
4. 프롬프트가 여전히 나타나지 않으면 `tccutil`로 TCC 항목을 초기화하고 다시 시도하세요.
5. 일부 권한은 macOS를 완전히 재시작한 후에만 다시 나타납니다.

초기화 예시 (필요에 따라 번들 ID 교체):

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

## 파일 및 폴더 권한 (Desktop/Documents/Downloads)

macOS는 터미널/백그라운드 프로세스에 대해 Desktop, Documents, Downloads를 제한할 수도 있습니다. 파일 읽기나 디렉토리 목록이 중단되면, 파일 작업을 수행하는 동일한 프로세스 컨텍스트에 접근 권한을 부여하세요 (예: Terminal/iTerm, LaunchAgent 실행 앱 또는 SSH 프로세스).

해결 방법: 폴더별 권한 부여를 피하려면 파일을 OpenClaw 워크스페이스 (`~/.openclaw/workspace`)로 이동하세요.

권한을 테스트하는 경우 항상 실제 인증서로 서명하세요. 임시 서명 빌드는 권한이 중요하지 않은 빠른 로컬 실행에만 허용됩니다.
