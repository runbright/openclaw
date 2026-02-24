# OpenClaw ACP 브리지

이 문서는 OpenClaw ACP (Agent Client Protocol) 브리지가 어떻게 작동하는지,
ACP 세션을 게이트웨이 세션에 어떻게 매핑하는지, IDE가 이를 어떻게 호출해야 하는지 설명합니다.

## 개요

`openclaw acp`는 stdio를 통해 ACP 에이전트를 노출하고 WebSocket을 통해
실행 중인 OpenClaw 게이트웨이로 프롬프트를 전달합니다. ACP 세션 ID를 게이트웨이
세션 키에 매핑하여 IDE가 동일한 에이전트 트랜스크립트에 재연결하거나
요청 시 리셋할 수 있습니다.

핵심 목표:

- 최소한의 ACP 표면적 (stdio, NDJSON).
- 재연결 시에도 안정적인 세션 매핑.
- 기존 게이트웨이 세션 스토어와 함께 작동 (list/resolve/reset).
- 안전한 기본값 (기본적으로 격리된 ACP 세션 키).

## 사용 방법

IDE 또는 도구가 Agent Client Protocol을 사용하며 OpenClaw 게이트웨이 세션을
구동하고 싶을 때 ACP를 사용하세요.

빠른 단계:

1. 게이트웨이를 실행합니다 (로컬 또는 원격).
2. 게이트웨이 대상을 설정합니다 (`gateway.remote.url` + 인증) 또는 플래그를 전달합니다.
3. IDE가 stdio를 통해 `openclaw acp`를 실행하도록 설정합니다.

설정 예시:

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

실행 예시:

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

## 에이전트 선택

ACP는 에이전트를 직접 선택하지 않습니다. 게이트웨이 세션 키로 라우팅합니다.

특정 에이전트를 대상으로 하려면 에이전트 범위 세션 키를 사용하세요:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

각 ACP 세션은 단일 게이트웨이 세션 키에 매핑됩니다. 하나의 에이전트가 여러
세션을 가질 수 있습니다. ACP는 키 또는 레이블을 재정의하지 않는 한 기본적으로
격리된 `acp:<uuid>` 세션을 사용합니다.

## Zed 에디터 설정

`~/.config/zed/settings.json`에 커스텀 ACP 에이전트를 추가합니다:

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

특정 게이트웨이 또는 에이전트를 대상으로 하려면:

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

## 실행 모델

- ACP 클라이언트가 `openclaw acp`를 생성하고 stdio를 통해 ACP 메시지를 주고받습니다.
- 브리지는 기존 인증 설정 (또는 CLI 플래그)을 사용하여 게이트웨이에 연결합니다.
- ACP `prompt`는 게이트웨이 `chat.send`로 변환됩니다.
- 게이트웨이 스트리밍 이벤트는 ACP 스트리밍 이벤트로 다시 변환됩니다.
- ACP `cancel`은 활성 실행에 대한 게이트웨이 `chat.abort`에 매핑됩니다.

## 세션 매핑

기본적으로 각 ACP 세션은 전용 게이트웨이 세션 키에 매핑됩니다:

- 재정의하지 않는 한 `acp:<uuid>`.

세션을 재정의하거나 재사용하는 두 가지 방법:

1. CLI 기본값

```bash
openclaw acp --session agent:main:main
openclaw acp --session-label "support inbox"
openclaw acp --reset-session
```

2. 세션별 ACP 메타데이터

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true,
    "requireExisting": false
  }
}
```

규칙:

- `sessionKey`: 직접 게이트웨이 세션 키.
- `sessionLabel`: 레이블로 기존 세션 확인.
- `resetSession`: 첫 사용 전에 키에 대한 새 트랜스크립트 생성.
- `requireExisting`: 키/레이블이 존재하지 않으면 실패.

### 세션 목록

ACP `listSessions`는 게이트웨이 `sessions.list`에 매핑되며 IDE 세션 선택기에
적합한 필터링된 요약을 반환합니다. `_meta.limit`으로 반환되는 세션 수를 제한할 수 있습니다.

## 프롬프트 변환

ACP 프롬프트 입력은 게이트웨이 `chat.send`로 변환됩니다:

- `text` 및 `resource` 블록은 프롬프트 텍스트가 됩니다.
- 이미지 MIME 유형의 `resource_link`는 첨부 파일이 됩니다.
- 작업 디렉토리를 프롬프트에 접두사로 추가할 수 있습니다 (기본 활성화,
  `--no-prefix-cwd`로 비활성화 가능).

게이트웨이 스트리밍 이벤트는 ACP `message` 및 `tool_call` 업데이트로 변환됩니다.
게이트웨이 터미널 상태는 중지 이유와 함께 ACP `done`에 매핑됩니다:

- `complete` -> `stop`
- `aborted` -> `cancel`
- `error` -> `error`

## 인증 + 게이트웨이 검색

`openclaw acp`는 CLI 플래그 또는 설정에서 게이트웨이 URL과 인증을 확인합니다:

- `--url` / `--token` / `--password`가 우선됩니다.
- 그 외에는 설정된 `gateway.remote.*` 설정을 사용합니다.

## 운영 참고

- ACP 세션은 브리지 프로세스 수명 동안 메모리에 저장됩니다.
- 게이트웨이 세션 상태는 게이트웨이 자체에서 영속됩니다.
- `--verbose`는 ACP/게이트웨이 브리지 이벤트를 stderr로 로깅합니다 (stdout은 사용하지 않음).
- ACP 실행은 취소할 수 있으며 활성 실행 ID는 세션별로 추적됩니다.

## 호환성

- ACP 브리지는 `@agentclientprotocol/sdk` (현재 0.13.x)를 사용합니다.
- `initialize`, `newSession`, `loadSession`, `prompt`, `cancel`, `listSessions`를
  구현하는 ACP 클라이언트와 함께 작동합니다.

## 테스트

- 단위 테스트: `src/acp/session.test.ts`가 실행 ID 수명주기를 다룹니다.
- 전체 게이트: `pnpm build && pnpm check && pnpm test && pnpm docs:build`.

## 관련 문서

- CLI 사용법: `docs/cli/acp.md`
- 세션 모델: `docs/concepts/session.md`
- 세션 관리 내부: `docs/reference/session-management-compaction.md`
