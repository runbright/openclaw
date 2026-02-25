# OpenClaw ACP 브릿지

이 문서는 OpenClaw ACP(에이전트 클라이언트 프로토콜, Agent Client Protocol) 브릿지의 작동 방식, ACP 세션과 게이트웨이 세션의 매핑 방법, 그리고 IDE에서 이를 호출하는 방법을 설명합니다.

## 개요
`openclaw acp`는 stdio(표준 입출력)를 통해 ACP 에이전트를 노출하고, WebSocket을 통해 실행 중인 OpenClaw 게이트웨이로 프롬프트를 전달합니다. 또한 ACP 세션 ID를 게이트웨이 세션 키와 연결하여 유지하므로, IDE가 재연결되더라도 동일한 대화 문맥을 이어가거나 필요 시 초기화할 수 있습니다.

**주요 목표:**
- 최소한의 ACP 접점 (stdio, NDJSON).
- 재연결 시에도 안정적인 세션 매핑 유지.
- 기존 게이트웨이 세션 저장소와 연동.
- 격리된 ACP 세션 키를 기본값으로 사용하는 안전한 설정.

## 사용 방법
IDE 또는 사용 중인 도구가 Agent Client Protocol을 지원하고, 이를 통해 OpenClaw 게이트웨이 세션을 제어하고 싶을 때 ACP를 사용합니다.

1. 게이트웨이(로컬 또는 원격)를 실행합니다.
2. 게이트웨이 대상(`gateway.remote.url` 및 인증 정보)을 구성하거나 명령줄 플래그로 전달합니다.
3. IDE가 stdio를 통해 `openclaw acp`를 실행하도록 설정합니다.

### 명령어 예시
```bash
# 게이트웨이 설정
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>

# ACP 브릿지 실행
openclaw acp --url wss://gateway-host:18789 --token <token>
```

## IDE 설정 예시 (Zed 에디터)
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

## 세션 관리
기본적으로 각 ACP 세션은 전용 게이트웨이 세션 키(`acp:<uuid>`)에 매핑됩니다. 특정 에이전트의 세션을 재사용하거나 이름을 지정하려면 `--session` 플래그를 사용하세요.

- `sessionKey`: 직접 게이트웨이 세션 키 지정.
- `resetSession`: 사용 전 기존 대화 내역 초기화.
- `requireExisting`: 기존 세션이 없는 경우 오류 발생.

---
상세한 명령어 도음말은 `openclaw acp --help`를 참조하거나 공식 문서의 CLI 가이드를 확인하세요.
---
