---
summary: "OpenClaw의 일반적인 설정 구성을 위한 스키마 기반 설정 예시 가이드"
read_when:
  - OpenClaw 설정 방법을 배울 때
  - 다양한 구성 예시가 필요할 때
  - OpenClaw를 처음 설정할 때
title: "설정 예시"
---

# 설정 예시 (Configuration Examples)

아래 예시들은 현재 OpenClaw 설정 스키마를 따릅니다. 각 필드에 대한 상세한 설명은 [설정 참조 문서](/gateway/configuration)를 확인하십시오.

## 빠른 시작

### 1. 최소 설정
가장 핵심적인 부분만 포함한 설정입니다. 특정 번호로부터의 WhatsApp 메시지만 허용합니다.

```json5
{
  "agent": { "workspace": "~/.openclaw/workspace" },
  "channels": { "whatsapp": { "allowFrom": ["+821012345678"] } }
}
```

### 2. 권장 시작 설정
봇의 이름과 아이덴티티, 그리고 기본 모델과 채널 정책을 포함한 추천 구성입니다.

```json5
{
  "identity": {
    "name": "클로디",
    "theme": "친절한 조수",
    "emoji": "🦞"
  },
  "agent": {
    "workspace": "~/.openclaw/workspace",
    "model": { "primary": "anthropic/claude-3-5-sonnet-latest" }
  },
  "channels": {
    "whatsapp": {
      "allowFrom": ["+821012345678"],
      "groups": { "*": { "requireMention": true } } // 그룹 채팅에선 멘션시에만 답변
    }
  }
}
```

## 주요 기능별 설정 예시

### 환경 변수 및 인증 관리
API 키 로테이션이나 기본 인증 프로필을 설정합니다.

```json5
{
  "env": {
    "OPENROUTER_API_KEY": "sk-or-..."
  },
  "auth": {
    "profiles": {
      "anthropic:work": { "provider": "anthropic", "mode": "api_key" },
      "openai:default": { "provider": "openai", "mode": "api_key" }
    },
    "order": {
      "anthropic": ["anthropic:work"],
      "openai": ["openai:default"]
    }
  }
}
```

### 세션 관리 및 자동 청소
대화 세션을 사용자별로 구분하고, 오래된 기록을 자동으로 관리합니다.

```json5
{
  "session": {
    "scope": "per-sender", // 발신자별로 개별 세션 유지
    "reset": {
      "mode": "daily", // 매일 새벽에 세션 초기화
      "atHour": 4
    },
    "maintenance": {
      "pruneAfter": "30d", // 30일 지난 기록 자동 정리
      "maxDiskBytes": "500mb" // 디스크 사용량 제한
    }
  }
}
```

### 채널별 허용 목록 (보안)
특정 사용자나 그룹에서만 에이전트를 사용할 수 있도록 제한합니다.

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "BOT_TOKEN_HERE",
      "allowFrom": ["123456789"] // 허용할 텔레그램 사용자 ID
    },
    "discord": {
      "enabled": true,
      "token": "DISCORD_TOKEN_HERE",
      "dm": { "allowFrom": ["9876543210"] }
    }
  }
}
```

### 샌드박싱 (보안 실행)
에이전트가 실행하는 코드를 격리된 환경(도커 등)에서 안전하게 실행합니다.

```json5
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "docker": {
          "image": "openclaw-sandbox:latest",
          "network": "none" // 인터넷 연결 차단
        }
      }
    }
  }
}
```

## 설정 팁

- **JSON5 활용**: `openclaw.json` 파일은 JSON5 형식을 지원하므로 주석(`//`)이나 마지막 쉼표(Trailing comma)를 사용할 수 있습니다.
- **경로 설정**: 모든 경로는 되도록 절대 경로를 사용하거나 `~`를 사용한 홈 디렉토리 기준 경로를 권장합니다.
- **다중 에이전트**: 여러 목적의 에이전트를 운영하려면 `agents.list`에 각각의 설정을 추가하여 관리할 수 있습니다.

상세한 각 필드 정의는 [게이트웨이 설정 참조](/gateway/configuration-reference)를 확인하십시오.
---
