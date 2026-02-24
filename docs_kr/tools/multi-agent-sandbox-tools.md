---
summary: "에이전트별 샌드박스 + 도구 제한, 우선순위 및 예제"
title: "멀티 에이전트 샌드박스 및 도구"
read_when: "멀티 에이전트 게이트웨이에서 에이전트별 샌드박싱 또는 에이전트별 도구 허용/거부 정책이 필요한 경우."
status: active
---

# 멀티 에이전트 샌드박스 및 도구 구성

## 개요

멀티 에이전트 설정에서 각 에이전트는 자체적으로 다음을 가질 수 있습니다:

- **샌드박스 구성** (`agents.list[].sandbox`가 `agents.defaults.sandbox`를 재정의)
- **도구 제한** (`tools.allow` / `tools.deny`, 그리고 `agents.list[].tools`)

이를 통해 서로 다른 보안 프로필로 여러 에이전트를 실행할 수 있습니다:

- 전체 접근 권한이 있는 개인 비서
- 제한된 도구를 가진 가족/업무 에이전트
- 샌드박스에서 실행되는 공개용 에이전트

`setupCommand`는 `sandbox.docker`(전역 또는 에이전트별) 아래에 속하며 컨테이너가
생성될 때 한 번 실행됩니다.

인증은 에이전트별입니다: 각 에이전트는 자체 `agentDir` 인증 저장소에서 읽습니다:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

자격 증명은 에이전트 간에 **공유되지 않습니다**. 에이전트 간에 `agentDir`을 재사용하지 마세요.
자격 증명을 공유하려면 `auth-profiles.json`을 다른 에이전트의 `agentDir`에 복사하세요.

샌드박싱이 런타임에서 어떻게 동작하는지는 [샌드박싱](/gateway/sandboxing)을 참조하세요.
"왜 차단되는지?" 디버깅은 [샌드박스 vs 도구 정책 vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated) 및 `openclaw sandbox explain`을 참조하세요.

---

## 구성 예제

### 예제 1: 개인 + 제한된 가족 에이전트

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**결과:**

- `main` 에이전트: 호스트에서 실행, 전체 도구 접근
- `family` 에이전트: Docker에서 실행(에이전트당 하나의 컨테이너), `read` 도구만 사용 가능

---

### 예제 2: 공유 샌드박스를 사용하는 업무 에이전트

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

---

### 예제 2b: 전역 코딩 프로필 + 메시징 전용 에이전트

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**결과:**

- 기본 에이전트는 코딩 도구를 받습니다
- `support` 에이전트는 메시징 전용입니다(+ Slack 도구)

---

### 예제 3: 에이전트별 다른 샌드박스 모드

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // 전역 기본값
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // 재정의: main은 절대 샌드박스되지 않음
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // 재정의: public은 항상 샌드박스됨
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

---

## 구성 우선순위

전역(`agents.defaults.*`)과 에이전트별(`agents.list[].*`) 구성이 모두 존재하는 경우:

### 샌드박스 구성

에이전트별 설정이 전역을 재정의합니다:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**참고:**

- `agents.list[].sandbox.{docker,browser,prune}.*`는 해당 에이전트에 대해 `agents.defaults.sandbox.{docker,browser,prune}.*`를 재정의합니다(샌드박스 범위가 `"shared"`로 해석되면 무시됨).

### 도구 제한

필터링 순서는 다음과 같습니다:

1. **도구 프로필** (`tools.profile` 또는 `agents.list[].tools.profile`)
2. **프로바이더 도구 프로필** (`tools.byProvider[provider].profile` 또는 `agents.list[].tools.byProvider[provider].profile`)
3. **전역 도구 정책** (`tools.allow` / `tools.deny`)
4. **프로바이더 도구 정책** (`tools.byProvider[provider].allow/deny`)
5. **에이전트별 도구 정책** (`agents.list[].tools.allow/deny`)
6. **에이전트 프로바이더 정책** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **샌드박스 도구 정책** (`tools.sandbox.tools` 또는 `agents.list[].tools.sandbox.tools`)
8. **서브에이전트 도구 정책** (`tools.subagents.tools`, 해당되는 경우)

각 수준은 도구를 추가로 제한할 수 있지만 이전 수준에서 거부된 도구를 다시 부여할 수는 없습니다.
`agents.list[].tools.sandbox.tools`가 설정되면 해당 에이전트의 `tools.sandbox.tools`를 대체합니다.
`agents.list[].tools.profile`이 설정되면 해당 에이전트의 `tools.profile`을 재정의합니다.
프로바이더 도구 키는 `provider`(예: `google-antigravity`) 또는 `provider/model`(예: `openai/gpt-5.2`)을 허용합니다.

### 도구 그룹(축약형)

도구 정책(전역, 에이전트, 샌드박스)은 여러 구체적인 도구로 확장되는 `group:*` 항목을 지원합니다:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: 모든 내장 OpenClaw 도구(프로바이더 플러그인 제외)

### Elevated 모드

`tools.elevated`는 전역 기준선(발신자 기반 허용 목록)입니다. `agents.list[].tools.elevated`는 특정 에이전트에 대해 elevated를 추가로 제한할 수 있습니다(양쪽 모두 허용해야 함).

완화 패턴:

- 신뢰할 수 없는 에이전트에 대해 `exec` 거부 (`agents.list[].tools.deny: ["exec"]`)
- 제한된 에이전트로 라우팅되는 발신자를 허용 목록에 추가하지 마세요
- 샌드박스 실행만 원하는 경우 전역적으로 elevated 비활성화 (`tools.elevated.enabled: false`)
- 민감한 프로필에 대해 에이전트별 elevated 비활성화 (`agents.list[].tools.elevated.enabled: false`)

---

## 단일 에이전트에서 마이그레이션

**이전(단일 에이전트):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**이후(다른 프로필을 가진 멀티 에이전트):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

레거시 `agent.*` 구성은 `openclaw doctor`에 의해 마이그레이션됩니다; 앞으로는 `agents.defaults` + `agents.list`를 사용하세요.

---

## 도구 제한 예제

### 읽기 전용 에이전트

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### 안전한 실행 에이전트(파일 수정 없음)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### 통신 전용 에이전트

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

---

## 자주 발생하는 함정: "non-main"

`agents.defaults.sandbox.mode: "non-main"`은 에이전트 id가 아니라 `session.mainKey`(기본값 `"main"`)를 기반으로 합니다.
그룹/채널 세션은 항상 자체 키를 가지므로 non-main으로 취급되어 샌드박스됩니다. 에이전트가 절대 샌드박스되지 않도록 하려면 `agents.list[].sandbox.mode: "off"`를 설정하세요.

---

## 테스트

멀티 에이전트 샌드박스 및 도구를 구성한 후:

1. **에이전트 해석 확인:**

   ```exec
   openclaw agents list --bindings
   ```

2. **샌드박스 컨테이너 확인:**

   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **도구 제한 테스트:**
   - 제한된 도구가 필요한 메시지를 보냅니다
   - 에이전트가 거부된 도구를 사용할 수 없는지 확인합니다

4. **로그 모니터링:**

   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

---

## 문제 해결

### `mode: "all"`에도 에이전트가 샌드박스되지 않는 경우

- 전역 `agents.defaults.sandbox.mode`가 이를 재정의하는지 확인하세요
- 에이전트별 구성이 우선하므로 `agents.list[].sandbox.mode: "all"`을 설정하세요

### 거부 목록에도 도구가 여전히 사용 가능한 경우

- 도구 필터링 순서를 확인하세요: 전역 → 에이전트 → 샌드박스 → 서브에이전트
- 각 수준은 추가로 제한만 할 수 있고 다시 부여할 수 없습니다
- 로그로 확인: `[tools] filtering tools for agent:${agentId}`

### 컨테이너가 에이전트별로 격리되지 않는 경우

- 에이전트별 샌드박스 구성에서 `scope: "agent"`를 설정하세요
- 기본값은 세션당 하나의 컨테이너를 만드는 `"session"`입니다

---

## 참조

- [멀티 에이전트 라우팅](/concepts/multi-agent)
- [샌드박스 구성](/gateway/configuration#agentsdefaults-sandbox)
- [세션 관리](/concepts/session)
