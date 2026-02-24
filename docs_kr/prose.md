---
summary: "OpenProse: OpenClaw에서의 .prose 워크플로우, 슬래시 명령어 및 상태 관리 안내"
read_when:
  - .prose 워크플로우를 작성하거나 실행하고 싶을 때
  - OpenProse 플러그인을 활성화하고 싶을 때
  - 상태 정보가 어디에 저장되는지 궁금할 때
title: "OpenProse"
---

# OpenProse

OpenProse는 AI 세션을 조율하기 위한 마크다운 기반의 이식성 높은 워크플로우 형식입니다. OpenClaw에서는 OpenProse 스킬 팩과 `/prose` 슬래시 명령어를 제공하는 플러그인 형태로 포함되어 있습니다. `.prose` 파일에 프로그램을 작성하면 명시적인 흐름 제어를 통해 여러 서브 에이전트를 생성하고 관리할 수 있습니다.

공식 사이트: [https://www.prose.md](https://www.prose.md)

## 주요 기능

- **멀티 에이전트 리서치**: 여러 에이전트를 병렬로 실행하여 리서치 및 종합을 수행합니다.
- **반복 가능한 워크플로우**: 코드 리뷰, 장애 대응 트리아지, 콘텐츠 파이프라인 등 승인이 필요한 작업을 규칙화합니다.
- **재사용성**: 지원되는 에이전트 런타임 어디서나 실행할 수 있는 `.prose` 프로그램을 작성할 수 있습니다.

## 설치 및 활성화

내장 플러그인은 기본적으로 비활성화되어 있습니다. 다음 명령어로 OpenProse를 활성화하세요:

```bash
openclaw plugins enable open-prose
```

플러그인 활성화 후 게이트웨이를 재시작해야 합니다.

## 슬래시 명령어

OpenProse는 사용자가 직접 호출할 수 있는 `/prose` 명령어를 등록합니다.

주요 명령어:
- `/prose run <파일명.prose>`: 로컬 프로젝트의 .prose 파일을 실행합니다.
- `/prose run <URL>`: 원격에 있는 .prose 프로그램을 가져와 실행합니다.
- `/prose examples`: 사용 가능한 예제 목록을 확인합니다.
- `/prose help`: 도움말을 봅니다.

## `.prose` 파일 예시

```prose
# 두 개의 에이전트를 병렬로 실행하여 조사 및 요약 수행

input topic: "무엇을 조사할까요?"

agent researcher:
  model: sonnet
  prompt: "철저하게 조사하고 출처를 밝히세요."

agent writer:
  model: opus
  prompt: "내용을 간결하게 요약하세요."

parallel:
  findings = session: researcher
    prompt: "{topic}에 대해 조사하세요."
  draft = session: writer
    prompt: "{topic}의 내용을 요약하세요."

session "{findings}와 {draft}의 내용을 합쳐서 최종 답변을 작성하세요."
```

## 파일 위치 및 상태 관리

OpenProse는 워크스페이스 내의 `.prose/` 디렉토리에 실행 상태를 보관합니다.
- `.prose/runs/`: 각 실행별 로그 및 상태 정보
- `~/.prose/agents/`: 사용자 수준의 영구 에이전트 설정

## 보안 안내

`.prose` 파일은 소스 코드와 다름없습니다. 실행하기 전에 내용을 검토하십시오. OpenClaw의 도구 허용 목록과 승인 절차를 활용하여 의도치 않은 시스템 변경이 발생하지 않도록 제어할 수 있습니다.
---
