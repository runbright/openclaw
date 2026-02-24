---
summary: "채팅 UI를 위한 루프백 WebChat 정적 호스트 및 Gateway WS 사용"
read_when:
  - WebChat 접근 디버깅 또는 구성 시
title: "WebChat"
---

# WebChat (Gateway WebSocket UI)

상태: macOS/iOS SwiftUI 채팅 UI가 Gateway WebSocket에 직접 통신합니다.

## 정의

- gateway를 위한 네이티브 채팅 UI입니다 (임베디드 브라우저 없고 로컬 정적 서버 없음).
- 다른 채널과 동일한 세션 및 라우팅 규칙을 사용합니다.
- 결정적 라우팅: 응답은 항상 WebChat으로 돌아갑니다.

## 빠른 시작

1. gateway를 시작합니다.
2. WebChat UI (macOS/iOS 앱) 또는 제어 UI 채팅 탭을 엽니다.
3. gateway 인증이 구성되어 있는지 확인합니다 (루프백에서도 기본적으로 필요).

## 작동 방식 (동작)

- UI가 Gateway WebSocket에 연결하고 `chat.history`, `chat.send`, `chat.inject`를 사용합니다.
- `chat.history`는 안정성을 위해 제한됩니다: Gateway가 긴 텍스트 필드를 잘라내고, 무거운 메타데이터를 생략하며, 초과 크기 항목을 `[chat.history omitted: message too large]`로 대체할 수 있습니다.
- `chat.inject`는 트랜스크립트에 직접 어시스턴트 노트를 추가하고 UI에 브로드캐스트합니다 (에이전트 실행 없음).
- 중단된 실행은 UI에서 부분 어시스턴트 출력을 표시 유지할 수 있습니다.
- Gateway는 버퍼된 출력이 있을 때 중단된 부분 어시스턴트 텍스트를 트랜스크립트 이력에 저장하며, 해당 항목에 중단 메타데이터를 표시합니다.
- 이력은 항상 gateway에서 가져옵니다 (로컬 파일 감시 없음).
- gateway에 접근할 수 없으면 WebChat은 읽기 전용입니다.

## 제어 UI 에이전트 도구 패널

- 제어 UI `/agents` 도구 패널은 `tools.catalog`를 통해 런타임 카탈로그를 가져오고 각
  도구에 `core` 또는 `plugin:<id>` (선택적 플러그인 도구에는 `optional`) 레이블을 붙입니다.
- `tools.catalog`를 사용할 수 없으면 패널은 내장 정적 목록으로 폴백합니다.
- 패널은 프로필 및 재정의 설정을 편집하지만, 유효 런타임 접근은 여전히 정책
  우선순위 (`allow`/`deny`, 에이전트별 및 프로바이더/채널 재정의)를 따릅니다.

## 원격 사용

- 원격 모드는 SSH/Tailscale을 통해 gateway WebSocket을 터널링합니다.
- 별도의 WebChat 서버를 실행할 필요가 없습니다.

## 설정 참조 (WebChat)

전체 설정: [설정](/gateway/configuration)

채널 옵션:

- 전용 `webchat.*` 블록 없음. WebChat은 아래의 gateway 엔드포인트 + 인증 설정을 사용합니다.

관련 글로벌 옵션:

- `gateway.port`, `gateway.bind`: WebSocket 호스트/포트.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 인증 (토큰/비밀번호).
- `gateway.auth.mode: "trusted-proxy"`: 브라우저 클라이언트를 위한 역방향 프록시 인증 ([신뢰할 수 있는 프록시 인증](/gateway/trusted-proxy-auth) 참조).
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: 원격 gateway 대상.
- `session.*`: 세션 저장소 및 메인 키 기본값.
