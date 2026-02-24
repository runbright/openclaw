---
summary: "CLI 백엔드: 로컬 AI CLI를 활용한 텍스트 전용 대체(Fallback) 실행 가이드"
read_when:
  - API 서비스 장애 시 신뢰할 수 있는 대체 수단이 필요할 때
  - Claude Code CLI 등 이미 설치된 로컬 AI 도구를 OpenClaw에서 재사용하고 싶을 때
  - 도구 사용 없이 순수 텍스트 대화만 필요한 안정적인 경로를 원할 때
title: "CLI 백엔드"
---

# CLI 백엔드 (CLI Backends)

OpenClaw는 API 제공업체의 서버 장애나 속도 제한 시 도움을 주는 **텍스트 전용 대체(Fallback) 실행기**로서 **로컬 AI CLI**를 호출할 수 있습니다. 이는 매우 안정적인 "최후의 보루" 역할을 하도록 설계되었습니다.

## 특징 및 제한 사항

- **순수 텍스트 통신**: 도구(Tools) 호출은 지원되지 않으며 텍스트 입력과 출력만 담당합니다.
- **세션 유지**: CLI 수준에서 세션을 지원하는 경우 대화의 맥락이 유지됩니다.
- **이미지 지원**: CLI가 이미지 경로를 인자로 받을 수 있다면 이미지 분석도 가능합니다.
- **오프라인 지향**: 외부 API 호출 없이 로컬에 설치된 CLI를 직접 실행하므로 매우 신뢰도가 높습니다.

## 빠른 시작 (Claude Code CLI 예시)

설정 없이도 바로 사용할 수 있는 빌트인 설정을 제공합니다:

```bash
# Claude Code CLI 사용
openclaw agent --message "안녕" --model claude-cli/opus-4.6

# Codex CLI 사용
openclaw agent --message "안녕" --model codex-cli/gpt-5.3-codex
```

만약 게이트웨이가 데몬(systemd/launchd)으로 실행 중이라 PATH 인식이 안 된다면, 설정 파일에 실제 명령어 경로를 지정해 주어야 합니다:

```json5
{
  "agents": {
    "defaults": {
      "cliBackends": {
        "claude-cli": {
          "command": "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

## 대체(Fallback) 모델로 설정하기

주요 모델이 실패했을 때 자동으로 로컬 CLI를 사용하도록 설정하는 것이 가장 추천되는 방식입니다:

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-3-5-sonnet",
        "fallbacks": ["claude-cli/opus-4.6"]
      }
    }
  }
}
```

## 작동 원리

1. **백엔드 선택**: 모델 참조 프로필(`claude-cli/...`)을 보고 실행할 CLI를 결정합니다.
2. **프롬프트 조립**: OpenClaw의 기본 시스템 프롬프트와 워크스페이스 컨텍스트를 해당 CLI 형식에 맞춰 변환합니다.
3. **CLI 실행**: 세션 ID를 함께 전달하여 대화 기록을 이어나갑니다.
4. **결과 파싱**: CLI의 출력(JSON, JSONL 또는 일반 텍스트)에서 답변 내용을 추출합니다.
5. **세션 유지**: 다음에 대화할 때도 동일한 CLI 세션 ID를 사용하여 대화를 재개합니다.

## 주요 설정 옵션

- **`command`**: 실행할 CLI 파일의 절대 경로 또는 이름.
- **`args`**: 실행 시 전달할 기본 인자들.
- **`modelAliases`**: OpenClaw의 모델 이름을 해당 CLI가 인식하는 이름으로 변환하는 매핑 테이블.
- **`sessionArg`**: 세션 ID를 전달하기 위한 인자 이름 (예: `--session-id`).
- **`output`**: 출력 형식 (`json`, `jsonl`, `text` 중 선택).

상세한 빌트인 설정 값이나 커스텀 CLI 연동 방법은 [영문 원본 문서](/gateway/cli-backends)를 참고하십시오.
---
