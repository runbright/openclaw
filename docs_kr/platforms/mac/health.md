---
summary: "macOS 앱이 게이트웨이/Baileys 상태를 보고하는 방법"
read_when:
  - macOS 앱 상태 표시 디버깅
title: "상태 확인"
---

# macOS 상태 확인

메뉴 바 앱에서 연결된 채널이 정상인지 확인하는 방법입니다.

## 메뉴 바

- 상태 점이 이제 Baileys 상태를 반영합니다:
  - 초록색: 연결됨 + 소켓이 최근에 열림.
  - 주황색: 연결 중/재시도 중.
  - 빨간색: 로그아웃됨 또는 프로브 실패.
- 보조 줄에 "linked · auth 12m"이 표시되거나 실패 이유가 표시됩니다.
- "Run Health Check" 메뉴 항목으로 요청 시 프로브를 트리거합니다.

## 설정

- General 탭에 다음을 보여주는 Health 카드가 추가됩니다: 연결 인증 기간, 세션 저장 경로/개수, 마지막 확인 시간, 마지막 오류/상태 코드, 그리고 Run Health Check / Reveal Logs 버튼.
- 캐시된 스냅샷을 사용하여 UI가 즉시 로드되며 오프라인 시 우아하게 대체됩니다.
- **Channels 탭**에서 WhatsApp/Telegram의 채널 상태 + 제어를 표시합니다 (로그인 QR, 로그아웃, 프로브, 마지막 연결 끊김/오류).

## 프로브 작동 방식

- 앱은 `ShellExecutor`를 통해 약 60초마다 및 요청 시 `openclaw health --json`을 실행합니다. 프로브는 자격 증명을 로드하고 메시지를 보내지 않고 상태를 보고합니다.
- 마지막 정상 스냅샷과 마지막 오류를 별도로 캐시하여 깜빡임을 방지합니다; 각각의 타임스탬프를 표시합니다.

## 확실하지 않을 때

- [게이트웨이 상태](/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`)의 CLI 흐름을 여전히 사용할 수 있으며, `/tmp/openclaw/openclaw-*.log`에서 `web-heartbeat` / `web-reconnect`를 tail할 수 있습니다.
