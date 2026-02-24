---
summary: "`openclaw completion` CLI 참조 (셸 자동 완성 스크립트 생성/설치)"
read_when:
  - zsh/bash/fish/PowerShell 셸 자동 완성이 필요한 경우
  - OpenClaw 상태 디렉토리에 자동 완성 스크립트를 캐시해야 하는 경우
title: "completion"
---

# `openclaw completion`

셸 자동 완성 스크립트를 생성하고 선택적으로 셸 프로필에 설치합니다.

## 사용법

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## 옵션

- `-s, --shell <shell>`: 셸 대상 (`zsh`, `bash`, `powershell`, `fish`; 기본값: `zsh`)
- `-i, --install`: 셸 프로필에 source 줄을 추가하여 자동 완성을 설치합니다
- `--write-state`: stdout에 출력하지 않고 `$OPENCLAW_STATE_DIR/completions`에 자동 완성 스크립트를 기록합니다
- `-y, --yes`: 설치 확인 프롬프트를 건너뜁니다

## 참고사항

- `--install`은 셸 프로필에 작은 "OpenClaw Completion" 블록을 작성하고 캐시된 스크립트를 가리킵니다.
- `--install` 또는 `--write-state` 없이는 스크립트를 stdout으로 출력합니다.
- 자동 완성 생성은 중첩된 하위 명령이 포함되도록 명령 트리를 즉시 로드합니다.
