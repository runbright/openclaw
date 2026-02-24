---
summary: "`openclaw models` (상태/목록/설정/스캔, 별칭, 폴백, 인증) CLI 레퍼런스"
read_when:
  - 기본 모델을 변경하거나 프로바이더 인증 상태를 확인하고 싶을 때
  - 사용 가능한 모델/프로바이더를 스캔하고 인증 프로필을 디버깅하고 싶을 때
title: "models"
---

# `openclaw models`

모델 검색, 스캔 및 구성 (기본 모델, 폴백, 인증 프로필).

관련 항목:

- 프로바이더 + 모델: [모델](/providers/models)
- 프로바이더 인증 설정: [시작하기](/start/getting-started)

## 일반 명령어

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status`는 해석된 기본값/폴백과 인증 개요를 표시합니다.
프로바이더 사용량 스냅샷이 사용 가능한 경우, OAuth/토큰 상태 섹션에
프로바이더 사용량 헤더가 포함됩니다.
`--probe`를 추가하면 구성된 각 프로바이더 프로필에 대해 라이브 인증 프로브를 실행합니다.
프로브는 실제 요청이므로 (토큰을 소비하고 속도 제한을 트리거할 수 있습니다).
`--agent <id>`를 사용하여 구성된 에이전트의 모델/인증 상태를 검사합니다. 생략하면
`OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`이 설정된 경우 이를 사용하고, 그렇지 않으면
구성된 기본 에이전트를 사용합니다.

참고:

- `models set <model-or-alias>`는 `provider/model` 또는 별칭을 허용합니다.
- 모델 참조는 **첫 번째** `/`를 기준으로 분할하여 파싱됩니다. 모델 ID에 `/`가 포함된 경우(OpenRouter 스타일), 프로바이더 접두사를 포함하세요 (예: `openrouter/moonshotai/kimi-k2`).
- 프로바이더를 생략하면 OpenClaw는 입력을 별칭 또는 **기본 프로바이더**의 모델로 처리합니다 (모델 ID에 `/`가 없는 경우에만 작동).

### `models status`

옵션:

- `--json`
- `--plain`
- `--check` (종료 코드 1=만료/누락, 2=만료 임박)
- `--probe` (구성된 인증 프로필의 라이브 프로브)
- `--probe-provider <name>` (하나의 프로바이더를 프로브)
- `--probe-profile <id>` (반복 또는 쉼표로 구분된 프로필 ID)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>` (구성된 에이전트 ID; `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`을 재정의)

## 별칭 + 폴백

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## 인증 프로필

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login`은 프로바이더 플러그인의 인증 플로우(OAuth/API 키)를 실행합니다.
`openclaw plugins list`를 사용하여 설치된 프로바이더를 확인하세요.

참고:

- `setup-token`은 setup-token 값을 입력하라는 프롬프트를 표시합니다 (모든 머신에서 `claude setup-token`으로 생성).
- `paste-token`은 다른 곳이나 자동화에서 생성된 토큰 문자열을 허용합니다.
