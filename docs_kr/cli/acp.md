---
summary: "IDE 통합을 위한 ACP 브릿지 실행"
read_when:
  - ACP 기반 IDE 통합을 설정할 때
  - ACP 세션 라우팅을 Gateway로 디버깅할 때
title: "acp"
---

# acp

Gateway와 통신하는 ACP (Agent Client Protocol) 브릿지를 실행합니다.

이 명령은 IDE를 위해 stdio를 통해 ACP를 사용하며, 프롬프트를 WebSocket을 통해 Gateway로 전달합니다. ACP 세션을 Gateway 세션 키에 매핑하여 유지합니다.

## 사용법

```bash
openclaw acp

# 원격 Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# 원격 Gateway (파일에서 토큰 읽기)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 기존 세션 키에 연결
openclaw acp --session agent:main:main

# 라벨로 연결 (이미 존재해야 함)
openclaw acp --session-label "support inbox"

# 첫 번째 프롬프트 전에 세션 키 초기화
openclaw acp --session agent:main:main --reset-session
```

## ACP 클라이언트 (디버그)

내장 ACP 클라이언트를 사용하여 IDE 없이 브릿지를 간단히 점검할 수 있습니다.
ACP 브릿지를 생성하고 대화형으로 프롬프트를 입력할 수 있습니다.

```bash
openclaw acp client

# 생성된 브릿지를 원격 Gateway로 지정
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 서버 명령 재정의 (기본값: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

권한 모델 (클라이언트 디버그 모드):

- 자동 승인은 허용 목록 기반이며, 신뢰할 수 있는 핵심 도구 ID에만 적용됩니다.
- `read` 자동 승인은 현재 작업 디렉토리(`--cwd` 설정 시)로 범위가 제한됩니다.
- 알 수 없는/비핵심 도구 이름, 범위 밖의 읽기, 위험한 도구는 항상 명시적 프롬프트 승인이 필요합니다.
- 서버에서 제공하는 `toolCall.kind`는 신뢰할 수 없는 메타데이터로 취급됩니다 (권한 부여 소스가 아님).

## 사용 방법

IDE(또는 기타 클라이언트)가 Agent Client Protocol을 사용하고 이를 통해 OpenClaw Gateway 세션을 구동하려는 경우 ACP를 사용합니다.

1. Gateway가 실행 중인지 확인합니다 (로컬 또는 원격).
2. Gateway 대상을 구성합니다 (설정 또는 플래그).
3. IDE에서 `openclaw acp`를 stdio를 통해 실행하도록 지정합니다.

설정 예시 (영구 저장):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

직접 실행 예시 (설정 파일에 기록하지 않음):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# 로컬 프로세스 보안을 위해 권장
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## 에이전트 선택

ACP는 에이전트를 직접 선택하지 않습니다. Gateway 세션 키를 통해 라우팅합니다.

에이전트 범위의 세션 키를 사용하여 특정 에이전트를 대상으로 지정합니다:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

각 ACP 세션은 단일 Gateway 세션 키에 매핑됩니다. 하나의 에이전트에 여러 세션이 있을 수 있으며, ACP는 키나 라벨을 재정의하지 않는 한 격리된 `acp:<uuid>` 세션을 기본값으로 사용합니다.

## Zed 에디터 설정

`~/.config/zed/settings.json`에 커스텀 ACP 에이전트를 추가합니다 (또는 Zed 설정 UI를 사용):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

특정 Gateway 또는 에이전트를 대상으로 지정하려면:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Zed에서 Agent 패널을 열고 "OpenClaw ACP"를 선택하여 스레드를 시작합니다.

## 세션 매핑

기본적으로 ACP 세션은 `acp:` 접두사가 붙은 격리된 Gateway 세션 키를 받습니다.
알려진 세션을 재사용하려면 세션 키 또는 라벨을 전달합니다:

- `--session <key>`: 특정 Gateway 세션 키를 사용합니다.
- `--session-label <label>`: 라벨로 기존 세션을 찾습니다.
- `--reset-session`: 해당 키에 대해 새로운 세션 ID를 생성합니다 (같은 키, 새 대화 기록).

ACP 클라이언트가 메타데이터를 지원하는 경우 세션별로 재정의할 수 있습니다:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

세션 키에 대해 자세히 알아보려면 [/concepts/session](/concepts/session)을 참조하세요.

## 옵션

- `--url <url>`: Gateway WebSocket URL (구성된 경우 gateway.remote.url이 기본값).
- `--token <token>`: Gateway 인증 토큰.
- `--token-file <path>`: 파일에서 Gateway 인증 토큰을 읽습니다.
- `--password <password>`: Gateway 인증 비밀번호.
- `--password-file <path>`: 파일에서 Gateway 인증 비밀번호를 읽습니다.
- `--session <key>`: 기본 세션 키.
- `--session-label <label>`: 찾을 기본 세션 라벨.
- `--require-existing`: 세션 키/라벨이 존재하지 않으면 실패합니다.
- `--reset-session`: 첫 사용 전에 세션 키를 초기화합니다.
- `--no-prefix-cwd`: 프롬프트에 작업 디렉토리를 접두사로 추가하지 않습니다.
- `--verbose, -v`: stderr로 상세 로깅.

보안 참고사항:

- `--token`과 `--password`는 일부 시스템에서 로컬 프로세스 목록에 노출될 수 있습니다.
- `--token-file`/`--password-file` 또는 환경 변수(`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`)를 사용하는 것이 좋습니다.

### `acp client` 옵션

- `--cwd <dir>`: ACP 세션의 작업 디렉토리.
- `--server <command>`: ACP 서버 명령 (기본값: `openclaw`).
- `--server-args <args...>`: ACP 서버에 전달되는 추가 인수.
- `--server-verbose`: ACP 서버의 상세 로깅을 활성화합니다.
- `--verbose, -v`: 상세 클라이언트 로깅.
