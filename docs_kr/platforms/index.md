---
summary: "지원 플랫폼 개요 (게이트웨이 및 동반 앱)"
read_when:
  - 지원되는 OS나 설치 경로를 찾고 있을 때
  - 게이트웨이를 어디서 실행할지 결정해야 할 때
title: "플랫폼"
---

# 지원 플랫폼 (Platforms)

OpenClaw 코어는 TypeScript로 작성되었으며 **Node.js**가 권장 실행 환경입니다.

게이트웨이(서버)는 여러 운영체제에서 작동하며, 각 기기를 보조 노드로 연결하기 위한 전용 앱(macOS 메뉴바 앱, iOS/Android 앱)이 제공됩니다.

## 운영체제별 가이드

- **macOS**: [macOS 설정 가이드](/platforms/macos)
- **iOS**: [iOS 노드 연결](/platforms/ios)
- **Android**: [Android 노드 연결](/platforms/android)
- **Windows**: [Windows(WSL2) 설치 가이드](/platforms/windows)
- **Linux**: [리눅스 및 라즈베리 파이 가이드](/platforms/linux)

## 클라우드 및 VPS 호스팅

에이전트를 24시간 실행하고 싶다면 클라우드 환경을 추천합니다.
- [VPS 호스팅 허브](/vps) (전체 목록)
- [Fly.io 설치법](/install/fly)
- [Hetzner (Docker) 설치법](/install/hetzner)

## 게이트웨이 서비스 설치

백그라운드에서 자동으로 실행되도록 설정하려면 다음 명령어 중 하나를 사용하세요:
- **추천**: `openclaw onboard --install-daemon` (초기 설정 마법사)
- **직접 설치**: `openclaw gateway install`
- **자동 점검 및 수정**: `openclaw doctor`

**운영체제별 서비스 명칭:**
- **macOS**: LaunchAgent (`bot.molt.gateway`)
- **Linux/WSL2**: systemd 사용자 서비스 (`openclaw-gateway.service`)

---
상세한 설치 과정은 [시작하기](/start/getting-started) 문서를 참고하세요.
---
