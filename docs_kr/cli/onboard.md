---
summary: "`openclaw onboard` (인터랙티브 온보딩 마법사) CLI 레퍼런스"
read_when:
  - 게이트웨이, 워크스페이스, 인증, 채널, 스킬에 대한 가이드 설정이 필요할 때
title: "onboard"
---

# `openclaw onboard`

인터랙티브 온보딩 마법사 (로컬 또는 원격 Gateway 설정).

## 관련 가이드

- CLI 온보딩 허브: [온보딩 마법사 (CLI)](/start/wizard)
- 온보딩 개요: [온보딩 개요](/start/onboarding-overview)
- CLI 온보딩 레퍼런스: [CLI 온보딩 레퍼런스](/start/wizard-cli-reference)
- CLI 자동화: [CLI 자동화](/start/wizard-cli-automation)
- macOS 온보딩: [온보딩 (macOS 앱)](/start/onboarding)

## 예시

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

비인터랙티브 커스텀 프로바이더:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key`는 비인터랙티브 모드에서 선택 사항입니다. 생략하면 온보딩이 `CUSTOM_API_KEY`를 확인합니다.

비인터랙티브 Z.AI 엔드포인트 선택:

참고: `--auth-choice zai-api-key`는 이제 키에 가장 적합한 Z.AI 엔드포인트를 자동 감지합니다 (`zai/glm-5`가 있는 일반 API를 선호).
특히 GLM Coding Plan 엔드포인트를 원하는 경우 `zai-coding-global` 또는 `zai-coding-cn`을 선택하세요.

```bash
# 프롬프트 없는 엔드포인트 선택
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# 기타 Z.AI 엔드포인트 선택:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

비인터랙티브 Mistral 예시:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

플로우 참고:

- `quickstart`: 최소한의 프롬프트, 게이트웨이 토큰 자동 생성.
- `manual`: 포트/바인드/인증에 대한 전체 프롬프트 (`advanced`의 별칭).
- 로컬 온보딩 DM 범위 동작: [CLI 온보딩 레퍼런스](/start/wizard-cli-reference#outputs-and-internals).
- 가장 빠른 첫 채팅: `openclaw dashboard` (Control UI, 채널 설정 불필요).
- 커스텀 프로바이더: 목록에 없는 호스팅 프로바이더를 포함하여 모든 OpenAI 또는 Anthropic 호환 엔드포인트에 연결합니다. 자동 감지하려면 Unknown을 사용하세요.

## 일반적인 후속 명령어

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json`은 비인터랙티브 모드를 의미하지 않습니다. 스크립트에는 `--non-interactive`를 사용하세요.
</Note>
