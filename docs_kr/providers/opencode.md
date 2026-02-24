---
summary: "OpenClaw에서 OpenCode Zen(큐레이션된 모델) 사용하기"
read_when:
  - 모델 접근을 위해 OpenCode Zen을 사용하고 싶을 때
  - 코딩에 적합한 큐레이션된 모델 목록을 원할 때
title: "OpenCode Zen"
---

# OpenCode Zen

OpenCode Zen은 OpenCode 팀이 코딩 에이전트를 위해 추천하는 **큐레이션된 모델 목록**입니다.
API 키와 `opencode` 프로바이더를 사용하는 선택적 호스팅 모델 접근 경로입니다.
Zen은 현재 베타 단계입니다.

## CLI 설정

```bash
openclaw onboard --auth-choice opencode-zen
# 또는 비대화형
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## 설정 스니펫

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## 참고 사항

- `OPENCODE_ZEN_API_KEY`도 지원됩니다.
- Zen에 로그인하고, 결제 정보를 추가한 후, API 키를 복사하세요.
- OpenCode Zen은 요청당 과금됩니다. 자세한 내용은 OpenCode 대시보드를 확인하세요.
