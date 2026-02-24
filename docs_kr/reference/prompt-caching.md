---
title: "프롬프트 캐싱"
summary: "프롬프트 캐싱 설정, 병합 순서, 프로바이더 동작, 튜닝 패턴"
read_when:
  - You want to reduce prompt token costs with cache retention
  - You need per-agent cache behavior in multi-agent setups
  - You are tuning heartbeat and cache-ttl pruning together
---

# 프롬프트 캐싱

이 페이지는 프롬프트 재사용과 토큰 비용에 영향을 미치는 모든 캐시 관련 설정을 다룹니다.

Anthropic 가격 세부사항은 다음을 참조하세요:
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## 주요 설정

### `cacheRetention` (모델 및 에이전트별)

모델 파라미터에 캐시 보존 설정:

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

에이전트별 오버라이드:

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

설정 병합 순서:

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (일치하는 에이전트 id; 키별로 오버라이드)

### 레거시 `cacheControlTtl`

레거시 값은 여전히 허용되며 매핑됩니다:

- `5m` -> `short`
- `1h` -> `long`

새 설정에는 `cacheRetention`을 선호하세요.

### `contextPruning.mode: "cache-ttl"`

캐시 TTL 윈도우 이후 오래된 도구 결과 컨텍스트를 정리하여 유휴 후 요청이 과도한 히스토리를 재캐싱하지 않도록 합니다.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

전체 동작은 [세션 프루닝](/concepts/session-pruning)을 참조하세요.

### 하트비트 캐시 워밍

하트비트는 캐시 윈도우를 따뜻하게 유지하고 유휴 간격 후 반복적인 캐시 쓰기를 줄일 수 있습니다.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

에이전트별 하트비트는 `agents.list[].heartbeat`에서 지원됩니다.

## 프로바이더 동작

### Anthropic (직접 API)

- `cacheRetention`이 지원됩니다.
- Anthropic API 키 인증 프로필에서, OpenClaw은 미설정 시 Anthropic 모델 참조에 `cacheRetention: "short"`를 시드합니다.

### Amazon Bedrock

- Anthropic Claude 모델 참조 (`amazon-bedrock/*anthropic.claude*`)는 명시적 `cacheRetention` 패스스루를 지원합니다.
- Anthropic이 아닌 Bedrock 모델은 런타임에 `cacheRetention: "none"`으로 강제됩니다.

### OpenRouter Anthropic 모델

`openrouter/anthropic/*` 모델 참조의 경우, OpenClaw은 프롬프트 캐시 재사용을 개선하기 위해 시스템/개발자 프롬프트 블록에 Anthropic `cache_control`을 주입합니다.

### 기타 프로바이더

프로바이더가 이 캐시 모드를 지원하지 않으면 `cacheRetention`은 효과가 없습니다.

## 튜닝 패턴

### 혼합 트래픽 (권장 기본값)

주 에이전트에 장기 기준선을 유지하고, 버스트성 알림 에이전트에서는 캐싱을 비활성화:

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### 비용 우선 기준선

- 기준선 `cacheRetention: "short"` 설정.
- `contextPruning.mode: "cache-ttl"` 활성화.
- 따뜻한 캐시가 유익한 에이전트에만 TTL 미만으로 하트비트 유지.

## 캐시 진단

OpenClaw은 임베디드 에이전트 실행을 위한 전용 캐시 추적 진단을 노출합니다.

### `diagnostics.cacheTrace` 설정

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # 선택 사항
    includeMessages: false # 기본값 true
    includePrompt: false # 기본값 true
    includeSystem: false # 기본값 true
```

기본값:

- `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
- `includeMessages`: `true`
- `includePrompt`: `true`
- `includeSystem`: `true`

### 환경 토글 (일회성 디버깅)

- `OPENCLAW_CACHE_TRACE=1`은 캐시 추적을 활성화합니다.
- `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl`은 출력 경로를 오버라이드합니다.
- `OPENCLAW_CACHE_TRACE_MESSAGES=0|1`은 전체 메시지 페이로드 캡처를 토글합니다.
- `OPENCLAW_CACHE_TRACE_PROMPT=0|1`은 프롬프트 텍스트 캡처를 토글합니다.
- `OPENCLAW_CACHE_TRACE_SYSTEM=0|1`은 시스템 프롬프트 캡처를 토글합니다.

### 검사할 내용

- 캐시 추적 이벤트는 JSONL이며 `session:loaded`, `prompt:before`, `stream:context`, `session:after`와 같은 단계별 스냅샷을 포함합니다.
- 턴별 캐시 토큰 영향은 `/usage full` 및 세션 사용량 요약 등 일반 사용량 표면에서 `cacheRead`와 `cacheWrite`를 통해 확인할 수 있습니다.

## 빠른 트러블슈팅

- 대부분의 턴에서 높은 `cacheWrite`: 불안정한 시스템 프롬프트 입력을 확인하고 모델/프로바이더가 캐시 설정을 지원하는지 확인하세요.
- `cacheRetention`이 효과 없음: 모델 키가 `agents.defaults.models["provider/model"]`과 일치하는지 확인하세요.
- 캐시 설정이 있는 Bedrock Nova/Mistral 요청: 예상대로 런타임에서 `none`으로 강제됩니다.

관련 문서:

- [Anthropic](/providers/anthropic)
- [토큰 사용량 및 비용](/reference/token-use)
- [세션 프루닝](/concepts/session-pruning)
- [게이트웨이 설정 레퍼런스](/gateway/configuration-reference)
