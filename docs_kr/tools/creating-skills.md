---
title: "커스텀 스킬 만들기"
summary: "SKILL.md로 커스텀 워크스페이스 스킬 빌드 및 테스트"
read_when:
  - 워크스페이스에서 새로운 커스텀 스킬을 만들고 있을 때
  - SKILL.md 기반 스킬을 위한 빠른 시작 워크플로우가 필요할 때
---

# 커스텀 스킬 만들기

OpenClaw는 쉽게 확장할 수 있도록 설계되었습니다. "스킬"은 어시스턴트에 새로운 기능을 추가하는 주요 방법입니다.

## 스킬이란?

스킬은 `SKILL.md` 파일 (LLM에 지침과 도구 정의를 제공)과 선택적으로 일부 스크립트 또는 리소스를 포함하는 디렉토리입니다.

## 단계별: 첫 번째 스킬

### 1. 디렉토리 생성

스킬은 워크스페이스에 위치하며, 일반적으로 `~/.openclaw/workspace/skills/`입니다. 스킬을 위한 새 폴더를 생성하세요:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. `SKILL.md` 정의

해당 디렉토리에 `SKILL.md` 파일을 생성합니다. 이 파일은 메타데이터를 위한 YAML 프론트매터와 지침을 위한 Markdown을 사용합니다.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. 도구 추가 (선택 사항)

프론트매터에서 커스텀 도구를 정의하거나 에이전트에게 기존 시스템 도구 (`bash` 또는 `browser` 등)를 사용하도록 지시할 수 있습니다.

### 4. OpenClaw 새로고침

에이전트에게 "스킬 새로고침"을 요청하거나 게이트웨이를 재시작하세요. OpenClaw가 새 디렉토리를 검색하고 `SKILL.md`를 인덱싱합니다.

## 모범 사례

- **간결하게**: 모델에게 _무엇을_ 할지 지시하세요, AI가 되는 방법이 아닙니다.
- **안전 우선**: 스킬이 `bash`를 사용하는 경우, 신뢰할 수 없는 사용자 입력으로부터의 임의 명령 주입을 프롬프트가 허용하지 않도록 하세요.
- **로컬에서 테스트**: `openclaw agent --message "use my new skill"`을 사용하여 테스트하세요.

## 공유 스킬

[ClawHub](https://clawhub.com)에서 스킬을 탐색하고 기여할 수도 있습니다.
