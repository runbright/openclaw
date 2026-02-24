---
summary: "설정 개요: 일반적인 작업, 빠른 설정 및 전체 참조 문서 링크"
read_when:
  - OpenClaw를 처음으로 설정할 때
  - 일반적인 설정 패턴을 찾고 있을 때
  - 특정 설정 섹션으로 이동하고 싶을 때
title: "설정 가이드"
---

# 설정 (Configuration)

OpenClaw는 `~/.openclaw/openclaw.json` 경로에서 **JSON5** 형식의 설정 파일을 읽어옵니다. (JSON5는 주석과 마지막 쉼표를 지원하는 향상된 JSON 형식입니다.)

파일이 없더라도 OpenClaw는 안전한 기본값으로 실행되지만, 다음과 같은 경우 설정을 추가하는 것이 좋습니다:
- 메신저 채널을 연결하고 봇과 대화할 수 있는 사람을 제한하고 싶을 때
- 사용할 AI 모델, 도구, 샌드박싱 환경 또는 자동화(크론 잡, 훅)를 설정하고 싶을 때
- 세션 유지 시간, 미디어 처리 방식, 네트워크 또는 UI 설정을 조정하고 싶을 때

전체 필드 목록은 [설정 참조 문서](/gateway/configuration-reference)를 확인하십시오.

> **처음이신가요?** `openclaw onboard` 명령어로 대화형 설정을 시작하거나, [설정 예시](/gateway/configuration-examples) 가이드를 참고하여 필요한 내용을 복사해서 사용하세요.

## 설정 수정 방법

설정을 수정하는 방법은 크게 네 가지가 있습니다:

1. **대화형 마법사**: `openclaw onboard` (전체 설정) 또는 `openclaw configure` (설정 변경 전용) 실행.
2. **CLI 명령어**: `openclaw config set agents.defaults.workspace "~/workspace"` 같이 단일 키 값을 수정.
3. **제어 UI (Control UI)**: 브라우저에서 [http://127.0.0.1:18789](http://127.0.0.1:18789)에 접속하여 **Config** 탭 사용.
4. **직접 수정**: `~/.openclaw/openclaw.json` 파일을 텍스트 에디터로 직접 편집.

## 설정의 엄격한 검증

OpenClaw는 스키마와 완벽하게 일치하는 설정만 허용합니다. 잘못된 키 이름이나 형식이 포함되면 **게이트웨이가 시작되지 않을 수 있습니다.**
- 설정 오류로 실행이 안 된다면 `openclaw doctor` 명령어로 문제를 진단하십시오.
- `openclaw doctor --fix` 명령어로 자동 복구를 시도할 수 있습니다.

## 주요 설정 테마

### 1. 채널 연결 (WhatsApp, Telegram 등)
`channels.<서비스명>` 섹션에 각 서비스의 토큰이나 권한 설정을 넣습니다.
```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "BOT_TOKEN",
      "dmPolicy": "allowlist", // 허용 목록 기반 접근 제어
      "allowFrom": ["tg:12345"]
    }
  }
}
```

### 2. 모델 선택
메인 모델과 서비스 장애 시 사용할 대체(Fallback) 모델을 지정합니다.
```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-3-5-sonnet",
        "fallbacks": ["openai/gpt-4o"]
      }
    }
  }
}
```

### 3. 세션 및 초기화
대화의 연속성과 초기화 정책을 관리합니다.
- `dmScope`: 세션을 공유할지(`main`), 사용자별로 분리할지(`per-peer`) 결정.
- `reset`: 매일 정해진 시간이나 일정 시간 응답이 없을 때 세션을 초기화하도록 설정.

### 4. 샌드박싱 및 보안
코드를 안전하게 실행하기 위해 도커(Docker) 기반의 격리 환경을 사용하도록 설정할 수 있습니다.

## 설정 파일 분할 ($include)
설정 내용이 너무 길어지면 `$include`를 사용하여 파일을 나눌 수 있습니다.
```json5
{
  "gateway": { "port": 18789 },
  "agents": { "$include": "./agents.json5" }
}
```

## 자동 적용 (Hot Reload)
OpenClaw 게이트웨이는 설정 파일의 변경을 실시간으로 감시합니다. 대부분의 설정(채널, 모델, 도구 등)은 파일을 저장하자마자 **즉시(무중단) 적용**됩니다. 포트 변경이나 보안 프로토콜 변경과 같은 핵심 인프라 설정의 경우에만 자동으로 게이트웨이가 재시작됩니다.

상세한 필드별 설명은 **[게이트웨이 설정 참조](/gateway/configuration-reference)**를 확인하십시오.
---
