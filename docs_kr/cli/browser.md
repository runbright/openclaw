---
summary: "`openclaw browser` CLI 참조 (프로필, 탭, 액션, 확장 프로그램 릴레이)"
read_when:
  - "`openclaw browser`를 사용하면서 일반 작업 예시가 필요한 경우"
  - 노드 호스트를 통해 다른 머신에서 실행 중인 브라우저를 제어하려는 경우
  - Chrome 확장 프로그램 릴레이를 사용하려는 경우 (도구 모음 버튼으로 연결/분리)
title: "browser"
---

# `openclaw browser`

OpenClaw의 브라우저 제어 서버를 관리하고 브라우저 액션을 실행합니다 (탭, 스냅샷, 스크린샷, 탐색, 클릭, 입력).

관련 항목:

- 브라우저 도구 + API: [Browser tool](/tools/browser)
- Chrome 확장 프로그램 릴레이: [Chrome extension](/tools/chrome-extension)

## 공통 플래그

- `--url <gatewayWsUrl>`: Gateway WebSocket URL (설정에서 기본값).
- `--token <token>`: Gateway 토큰 (필요한 경우).
- `--timeout <ms>`: 요청 시간 제한 (ms).
- `--browser-profile <name>`: 브라우저 프로필 선택 (설정에서 기본값).
- `--json`: 기계 판독 가능 출력 (지원되는 경우).

## 빠른 시작 (로컬)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## 프로필

프로필은 이름이 지정된 브라우저 라우팅 설정입니다. 실제로는:

- `openclaw`: 전용 OpenClaw 관리 Chrome 인스턴스를 시작/연결합니다 (격리된 사용자 데이터 디렉토리).
- `chrome`: Chrome 확장 프로그램 릴레이를 통해 기존 Chrome 탭을 제어합니다.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

특정 프로필 사용:

```bash
openclaw browser --browser-profile work tabs
```

## 탭

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## 스냅샷 / 스크린샷 / 액션

스냅샷:

```bash
openclaw browser snapshot
```

스크린샷:

```bash
openclaw browser screenshot
```

탐색/클릭/입력 (ref 기반 UI 자동화):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome 확장 프로그램 릴레이 (도구 모음 버튼으로 연결)

이 모드에서는 에이전트가 수동으로 연결한 기존 Chrome 탭을 제어할 수 있습니다 (자동 연결되지 않음).

안정적인 경로에 언패킹된 확장 프로그램을 설치합니다:

```bash
openclaw browser extension install
openclaw browser extension path
```

그런 다음 Chrome -> `chrome://extensions` -> "개발자 모드" 활성화 -> "압축해제된 확장 프로그램을 로드합니다" -> 출력된 폴더를 선택합니다.

전체 가이드: [Chrome extension](/tools/chrome-extension)

## 원격 브라우저 제어 (노드 호스트 프록시)

Gateway가 브라우저와 다른 머신에서 실행되는 경우, Chrome/Brave/Edge/Chromium이 있는 머신에서 **노드 호스트**를 실행합니다. Gateway가 해당 노드로 브라우저 액션을 프록시합니다 (별도의 브라우저 제어 서버가 필요 없음).

`gateway.nodes.browser.mode`를 사용하여 자동 라우팅을 제어하고, 여러 노드가 연결된 경우 `gateway.nodes.browser.node`를 사용하여 특정 노드를 지정합니다.

보안 + 원격 설정: [Browser tool](/tools/browser), [Remote access](/gateway/remote), [Tailscale](/gateway/tailscale), [Security](/gateway/security)
