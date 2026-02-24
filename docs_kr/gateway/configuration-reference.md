---
title: "설정 레퍼런스"
description: "~/.openclaw/openclaw.json의 모든 항목에 대한 전체 레퍼런스"
summary: "OpenClaw의 모든 설정 키, 기본값, 채널 설정 등에 대한 기술적 상세 안내"
read_when:
  - 특정 설정 필드의 정확한 의미나 기본값이 궁금할 때
  - 채널, 모델, 게이트웨이 또는 도구 설정을 검증하고 싶을 때
---

# 설정 레퍼런스 (Configuration Reference)

이 문서는 `~/.openclaw/openclaw.json` 파일에서 사용할 수 있는 모든 설정 필드를 설명합니다. 일반적인 설정 가이드는 [설정 개요](/gateway/configuration)를 참고하세요.

설정 파일은 **JSON5** 형식을 따릅니다(주석 및 마지막 콤마 허용). 모든 필드는 선택 사항이며, 생략할 경우 OpenClaw의 기본값이 적용됩니다.

---

## 1. 채널 (Channels)

각 채널 설정이 존재하면 게이트웨이 시작 시 자동으로 활성화됩니다.

### DM 및 그룹 접근 정책
모든 채널은 다음과 같은 공통 정책을 지원합니다.

| DM 정책 | 동작 |
| :--- | :--- |
| `pairing` (기본값) | 모르는 사용자가 대화를 걸면 페어링 코드를 보내며, 관리자 승인이 필요합니다. |
| `allowlist` | `allowFrom` 목록에 있는 사용자만 허용합니다. |
| `open` | 모든 DM을 허용합니다 (`allowFrom: ["*"]` 설정 필수). |
| `disabled` | 모든 DM을 무시합니다. |

| 그룹 정책 | 동작 |
| :--- | :--- |
| `allowlist` (기본값) | 허용 목록에 있는 그룹에서만 동작합니다. |
| `open` | 모든 그룹에서 동작합니다 (멘션 시에만 응답). |
| `disabled` | 모든 그룹/방 메시지를 차단합니다. |

### WhatsApp 설정 예시
```json5
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "pairing",
      "allowFrom": ["+821012345678"],
      "sendReadReceipts": true, // 읽음 확인(파란 체크) 표시 여부
      "groups": {
        "*": { "requireMention": true } // 모든 그룹에서 @멘션 시에만 응답
      }
    }
  }
}
```

---

## 2. 에이전트 기본 설정 (Agent Defaults)

에이전트의 작동 방식, 모델 선택, 샌드박스 등을 설정합니다.

### 모델 설정 (`agents.defaults.model`)
주 모델과 장애 시 사용할 복구 모델(Fallback)을 지정합니다.

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6", // 주 모델
        "fallbacks": ["openai/gpt-5.2"] // 주 모델 실패 시 사용
      },
      "imageModel": {
        "primary": "google/gemini-2.0-flash-vision" // 이미지 분석용 모델
      }
    }
  }
}
```

### 샌드박싱 (`agents.defaults.sandbox`)
도구 실행을 격리된 도커 컨테이너에서 수행할지 설정합니다.
- `mode`: `off`(비활성), `non-main`(비주요 세션만), `all`(전체)
- `workspaceAccess`: `none`(접근 불가), `ro`(읽기 전용), `rw`(읽고 쓰기)

---

## 3. 도구 설정 (Tools)

에이전트가 사용할 수 있는 도구들을 제어합니다.

### 도구 프로필 (Profiles)
미리 정의된 도구 묶음을 선택할 수 있습니다.
- `minimal`: 상태 확인만 가능
- `messaging`: 메시지 전달 및 세션 관리 위주
- `coding`: 파일 수정, 명령어 실행 등 개발용 도구 포함
- `full`: 모든 도구 허용

### 도구 허용/차단 (`tools.allow` / `tools.deny`)
```json5
{
  "tools": {
    "deny": ["browser", "canvas"] // 특정 도구만 골라서 차단
  }
}
```

---

## 4. 게이트웨이 설정 (Gateway)

게이트웨이 서비스의 네트워크 및 인증을 설정합니다.

```json5
{
  "gateway": {
    "mode": "local",
    "port": 18789,
    "bind": "loopback", // 'loopback' 또는 'lan'
    "auth": {
      "mode": "token",
      "token": "여러분의_보안_토큰"
    },
    "tailscale": {
      "mode": "serve" // Tailscale Serve 자동 설정 활성화
    }
  }
}
```

---

## 5. 자동화 및 훅 (Hooks & Cron)

외부 알림을 받거나(Hooks) 주기적인 작업을 수행(Cron)합니다.

### 크론 설정 (`cron`)
```json5
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 2, // 동시에 실행될 최대 작업 수
    "sessionRetention": "24h" // 작업 완료 후 로그 보관 시간
  }
}
```

---

## 6. 환경 변수 (`env`)

설정 파일 내에서 `${VAR_NAME}` 형식으로 환경 변수를 참조할 수 있으며, 직접 값을 정의할 수도 있습니다.

```json5
{
  "env": {
    "vars": {
      "MY_API_KEY": "sk-..."
    }
  }
}
```

---

## 7. 설정 파일 분할 (`$include`)

설정이 너무 길어지면 파일을 나눌 수 있습니다.
```json5
{
  "gateway": { "port": 18789 },
  "agents": { "$include": "./agents-list.json5" }
}
```

---

**참고**: 이 문서는 전체 레퍼런스의 요약본입니다. 개별 필드에 대한 아주 상세한 기술적 정의나 모든 가능한 옵션 값은 [영문 원본 문서](/gateway/configuration-reference)를 확인해 주시기 바랍니다.
---
