---
summary: "macOS 앱이 게이트웨이 WebChat을 임베딩하는 방법과 디버깅 방법"
read_when:
  - macOS WebChat 뷰 또는 loopback 포트 디버깅
title: "WebChat"
---

# WebChat (macOS 앱)

macOS 메뉴 바 앱은 WebChat UI를 네이티브 SwiftUI 뷰로 임베딩합니다. 선택된 에이전트의 **메인 세션**에 연결되며 (다른 세션을 위한 세션 전환기 포함) 게이트웨이에 연결합니다.

- **로컬 모드**: 로컬 게이트웨이 WebSocket에 직접 연결합니다.
- **원격 모드**: SSH를 통해 게이트웨이 제어 포트를 전달하고 해당 터널을 데이터 플레인으로 사용합니다.

## 실행 및 디버깅

- 수동: Lobster 메뉴 → "Open Chat".
- 테스트용 자동 열기:

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- 로그: `./scripts/clawlog.sh` (서브시스템 `bot.molt`, 카테고리 `WebChatSwiftUI`).

## 연결 방식

- 데이터 플레인: 게이트웨이 WS 메서드 `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` 및 이벤트 `chat`, `agent`, `presence`, `tick`, `health`.
- 세션: 기본 세션 (`main`, 또는 스코프가 글로벌일 때 `global`)을 기본값으로 합니다. UI에서 세션을 전환할 수 있습니다.
- 온보딩은 첫 실행 설정을 분리하기 위해 전용 세션을 사용합니다.

## 보안 인터페이스

- 원격 모드는 SSH를 통해 게이트웨이 WebSocket 제어 포트만 전달합니다.

## 알려진 제한사항

- UI는 채팅 세션에 최적화되어 있습니다 (전체 브라우저 샌드박스가 아님).
