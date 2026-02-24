---
summary: "OpenClaw에서 API 키 또는 setup-token을 사용하여 Anthropic Claude 이용하기"
read_when:
  - You want to use Anthropic models in OpenClaw
  - You want setup-token instead of API keys
title: "Anthropic"
---

# Anthropic (Claude)

Anthropic은 **Claude** 모델 패밀리를 개발하며 API를 통해 접근을 제공합니다.
OpenClaw에서는 API 키 또는 **setup-token**으로 인증할 수 있습니다.

## 옵션 A: Anthropic API 키

**추천 대상:** 표준 API 접근 및 사용량 기반 과금.
Anthropic Console에서 API 키를 생성하세요.

### CLI 설정

```bash
openclaw onboard
# choose: Anthropic API key

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 설정 예시

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 프롬프트 캐싱 (Anthropic API)

OpenClaw은 Anthropic의 프롬프트 캐싱 기능을 지원합니다. 이 기능은 **API 전용**이며, 구독 인증에서는 캐시 설정이 적용되지 않습니다.

### 설정

모델 설정에서 `cacheRetention` 매개변수를 사용하세요:

| 값      | 캐시 지속 시간 | 설명                                |
| ------- | -------------- | ----------------------------------- |
| `none`  | 캐싱 없음      | 프롬프트 캐싱 비활성화              |
| `short` | 5분            | API 키 인증의 기본값                |
| `long`  | 1시간          | 확장 캐시 (베타 플래그 필요)        |

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### 기본값

Anthropic API 키 인증을 사용할 때, OpenClaw은 모든 Anthropic 모델에 자동으로 `cacheRetention: "short"` (5분 캐시)를 적용합니다. 설정에서 `cacheRetention`을 명시적으로 설정하여 이를 재정의할 수 있습니다.

### 에이전트별 cacheRetention 재정의

모델 수준 params를 기본값으로 사용하고, `agents.list[].params`를 통해 특정 에이전트를 재정의하세요.

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // baseline for most agents
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // override for this agent only
    ],
  },
}
```

캐시 관련 params의 설정 병합 순서:

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (일치하는 `id`, 키별로 재정의)

이를 통해 한 에이전트는 장기 캐시를 유지하면서, 같은 모델의 다른 에이전트는 버스트/저재사용 트래픽에서 쓰기 비용을 피하기 위해 캐싱을 비활성화할 수 있습니다.

### Bedrock Claude 참고사항

- Bedrock의 Anthropic Claude 모델 (`amazon-bedrock/*anthropic.claude*`)은 설정 시 `cacheRetention` 패스스루를 허용합니다.
- Bedrock의 비 Anthropic 모델은 런타임에 `cacheRetention: "none"`으로 강제됩니다.
- Anthropic API 키 스마트 기본값은 명시적 값이 설정되지 않은 경우 Claude-on-Bedrock 모델 참조에도 `cacheRetention: "short"`를 적용합니다.

### 레거시 매개변수

이전 `cacheControlTtl` 매개변수는 하위 호환성을 위해 계속 지원됩니다:

- `"5m"`은 `short`에 매핑됩니다
- `"1h"`은 `long`에 매핑됩니다

새로운 `cacheRetention` 매개변수로 마이그레이션하는 것을 권장합니다.

OpenClaw은 Anthropic API 요청에 `extended-cache-ttl-2025-04-11` 베타 플래그를 포함합니다.
프로바이더 헤더를 재정의하는 경우 이를 유지하세요 ([/gateway/configuration](/gateway/configuration) 참조).

## 1M 컨텍스트 윈도우 (Anthropic 베타)

Anthropic의 1M 컨텍스트 윈도우는 베타 게이트가 적용되어 있습니다. OpenClaw에서 지원되는 Opus/Sonnet 모델에
`params.context1m: true`를 설정하여 모델별로 활성화하세요.

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw은 이를 Anthropic 요청에서 `anthropic-beta: context-1m-2025-08-07`로 매핑합니다.

참고: Anthropic은 현재 OAuth/구독 토큰 (`sk-ant-oat-*`) 사용 시 `context-1m-*` 베타 요청을 거부합니다.
OpenClaw은 OAuth 인증 시 context1m 베타 헤더를 자동으로 건너뛰고 필요한 OAuth 베타만 유지합니다.

## 옵션 B: Claude setup-token

**추천 대상:** Claude 구독을 사용하는 경우.

### setup-token 얻는 방법

setup-token은 Anthropic Console이 아닌 **Claude Code CLI**에서 생성됩니다. **어떤 머신에서든** 실행할 수 있습니다:

```bash
claude setup-token
```

토큰을 OpenClaw에 붙여넣기 (마법사: **Anthropic token (paste setup-token)**) 하거나, 게이트웨이 호스트에서 실행하세요:

```bash
openclaw models auth setup-token --provider anthropic
```

다른 머신에서 토큰을 생성한 경우, 붙여넣기하세요:

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI 설정 (setup-token)

```bash
# Paste a setup-token during onboarding
openclaw onboard --auth-choice setup-token
```

### 설정 예시 (setup-token)

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 참고사항

- `claude setup-token`으로 setup-token을 생성하여 붙여넣거나, 게이트웨이 호스트에서 `openclaw models auth setup-token`을 실행하세요.
- Claude 구독에서 "OAuth token refresh failed ..."가 표시되면, setup-token으로 재인증하세요. [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription)을 참조하세요.
- 인증 세부사항 및 재사용 규칙은 [/concepts/oauth](/concepts/oauth)에 있습니다.

## 문제 해결

**401 오류 / 토큰이 갑자기 무효화됨**

- Claude 구독 인증이 만료되거나 취소될 수 있습니다. `claude setup-token`을 다시 실행하고
  **게이트웨이 호스트**에 붙여넣으세요.
- Claude CLI 로그인이 다른 머신에 있는 경우, 게이트웨이 호스트에서
  `openclaw models auth paste-token --provider anthropic`을 사용하세요.

**"anthropic" 프로바이더에 대한 API 키를 찾을 수 없음**

- 인증은 **에이전트별**입니다. 새 에이전트는 메인 에이전트의 키를 상속하지 않습니다.
- 해당 에이전트에 대해 온보딩을 다시 실행하거나, 게이트웨이 호스트에 setup-token / API 키를 붙여넣은 후
  `openclaw models status`로 확인하세요.

**프로필 `anthropic:default`에 대한 자격 증명을 찾을 수 없음**

- `openclaw models status`를 실행하여 활성 인증 프로필을 확인하세요.
- 온보딩을 다시 실행하거나, 해당 프로필에 setup-token / API 키를 붙여넣으세요.

**사용 가능한 인증 프로필 없음 (모두 쿨다운/사용 불가 상태)**

- `openclaw models status --json`에서 `auth.unusableProfiles`를 확인하세요.
- 다른 Anthropic 프로필을 추가하거나 쿨다운이 끝날 때까지 기다리세요.

자세한 내용: [/gateway/troubleshooting](/gateway/troubleshooting) 및 [/help/faq](/help/faq).
