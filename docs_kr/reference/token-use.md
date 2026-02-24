---
summary: "OpenClaw이 프롬프트 컨텍스트를 구성하고 토큰 사용량 + 비용을 보고하는 방법"
read_when:
  - Explaining token usage, costs, or context windows
  - Debugging context growth or compaction behavior
title: "토큰 사용량 및 비용"
---

# 토큰 사용량 및 비용

OpenClaw은 문자가 아닌 **토큰**을 추적합니다. 토큰은 모델별로 다르지만, 대부분의
OpenAI 스타일 모델은 영어 텍스트에서 토큰당 평균 약 4자입니다.

## 시스템 프롬프트 구성 방법

OpenClaw은 매 실행마다 자체 시스템 프롬프트를 조립합니다. 포함 내용:

- 도구 목록 + 짧은 설명
- 스킬 목록 (메타데이터만; 지침은 `read`로 필요시 로드)
- 자체 업데이트 지침
- 워크스페이스 + 부트스트랩 파일 (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, 새로운 경우 `BOOTSTRAP.md`, 존재 시 `MEMORY.md` 및/또는 `memory.md`). 큰 파일은 `agents.defaults.bootstrapMaxChars` (기본값: 20000)로 잘리며, 전체 부트스트랩 주입은 `agents.defaults.bootstrapTotalMaxChars` (기본값: 150000)로 제한됩니다. `memory/*.md` 파일은 메모리 도구를 통해 온디맨드이며 자동 주입되지 않습니다.
- 시간 (UTC + 사용자 시간대)
- 응답 태그 + 하트비트 동작
- 런타임 메타데이터 (호스트/OS/모델/사고)

전체 분석은 [시스템 프롬프트](/concepts/system-prompt)를 참조하세요.

## 컨텍스트 윈도우에 포함되는 것

모델이 받는 모든 것이 컨텍스트 제한에 포함됩니다:

- 시스템 프롬프트 (위에 나열된 모든 섹션)
- 대화 히스토리 (사용자 + 어시스턴트 메시지)
- 도구 호출 및 도구 결과
- 첨부파일/트랜스크립트 (이미지, 오디오, 파일)
- 컴팩션 요약 및 프루닝 아티팩트
- 프로바이더 래퍼 또는 안전 헤더 (보이지 않지만 여전히 카운트)

이미지의 경우, OpenClaw은 프로바이더 호출 전에 트랜스크립트/도구 이미지 페이로드를 축소합니다.
이를 튜닝하려면 `agents.defaults.imageMaxDimensionPx` (기본값: `1200`)를 사용하세요:

- 낮은 값은 일반적으로 비전 토큰 사용량과 페이로드 크기를 줄입니다.
- 높은 값은 OCR/UI 중심 스크린샷을 위해 더 많은 시각적 디테일을 보존합니다.

주입된 파일별, 도구, 스킬, 시스템 프롬프트 크기의 실용적 분석은 `/context list` 또는 `/context detail`을 사용하세요. [컨텍스트](/concepts/context)를 참조하세요.

## 현재 토큰 사용량 확인 방법

채팅에서 사용:

- `/status` → 세션 모델, 컨텍스트 사용량, 마지막 응답 입력/출력 토큰, **예상 비용** (API 키 전용)이 포함된 **상태 카드**.
- `/usage off|tokens|full` → 모든 응답에 **응답별 사용량 푸터**를 추가합니다.
  - 세션별로 유지됩니다 (`responseUsage`로 저장).
  - OAuth 인증은 **비용을 숨깁니다** (토큰만).
- `/usage cost` → OpenClaw 세션 로그에서 로컬 비용 요약을 표시합니다.

기타 표면:

- **TUI/Web TUI:** `/status` + `/usage`가 지원됩니다.
- **CLI:** `openclaw status --usage`와 `openclaw channels list`는
  프로바이더 할당량 윈도우 (응답별 비용이 아님)를 표시합니다.

## 비용 추정 (표시 시)

비용은 모델 가격 설정에서 추정됩니다:

```
models.providers.<provider>.models[].cost
```

이것은 `input`, `output`, `cacheRead`, `cacheWrite`에 대한 **100만 토큰당 USD**입니다.
가격이 없으면 OpenClaw은 토큰만 표시합니다. OAuth 토큰은
달러 비용을 표시하지 않습니다.

## 캐시 TTL 및 프루닝 영향

프로바이더 프롬프트 캐싱은 캐시 TTL 윈도우 내에서만 적용됩니다. OpenClaw은
선택적으로 **cache-ttl 프루닝**을 실행할 수 있습니다: 캐시 TTL이
만료된 후 세션을 프루닝하고, 캐시 윈도우를 리셋하여 후속 요청이 전체 히스토리를 재캐싱하는 대신
새로 캐시된 컨텍스트를 재사용할 수 있도록 합니다. 이렇게 하면 세션이 TTL 이후 유휴 상태일 때
캐시 쓰기 비용이 낮아집니다.

[게이트웨이 설정](/gateway/configuration)에서 설정하고
[세션 프루닝](/concepts/session-pruning)에서 동작 세부사항을 확인하세요.

하트비트는 유휴 간격에서 캐시를 **따뜻하게** 유지할 수 있습니다. 모델 캐시 TTL이
`1h`이면, 하트비트 간격을 그 바로 아래 (예: `55m`)로 설정하면 전체 프롬프트의
재캐싱을 방지하여 캐시 쓰기 비용을 줄일 수 있습니다.

멀티 에이전트 설정에서는 하나의 공유 모델 설정을 유지하고
`agents.list[].params.cacheRetention`으로 에이전트별 캐시 동작을 튜닝할 수 있습니다.

설정별 전체 가이드는 [프롬프트 캐싱](/reference/prompt-caching)을 참조하세요.

Anthropic API 가격의 경우, 캐시 읽기는 입력 토큰보다 상당히 저렴하고,
캐시 쓰기는 더 높은 승수로 청구됩니다. Anthropic의
프롬프트 캐싱 가격 최신 요율 및 TTL 승수는 다음을 참조하세요:
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### 예시: 하트비트로 1시간 캐시 따뜻하게 유지

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### 예시: 에이전트별 캐시 전략의 혼합 트래픽

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # 대부분 에이전트의 기본 기준선
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # 심층 세션을 위해 긴 캐시를 따뜻하게 유지
    - id: "alerts"
      params:
        cacheRetention: "none" # 버스트성 알림에 대한 캐시 쓰기 방지
```

`agents.list[].params`는 선택된 모델의 `params` 위에 병합되므로,
`cacheRetention`만 오버라이드하고 다른 모델 기본값은 변경 없이 상속할 수 있습니다.

### 예시: Anthropic 1M 컨텍스트 베타 헤더 활성화

Anthropic의 1M 컨텍스트 윈도우는 현재 베타 게이트됩니다. OpenClaw은
지원되는 Opus 또는 Sonnet 모델에서 `context1m`을 활성화하면 필요한 `anthropic-beta` 값을
주입할 수 있습니다.

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

이것은 Anthropic의 `context-1m-2025-08-07` 베타 헤더에 매핑됩니다.

Anthropic을 OAuth/구독 토큰 (`sk-ant-oat-*`)으로 인증하면,
OpenClaw은 Anthropic이 현재 해당 조합을 HTTP 401로
거부하기 때문에 `context-1m-*` 베타 헤더를 건너뜁니다.

## 토큰 압력 줄이기 팁

- `/compact`를 사용하여 긴 세션을 요약하세요.
- 워크플로우에서 큰 도구 출력을 줄이세요.
- 스크린샷이 많은 세션에서는 `agents.defaults.imageMaxDimensionPx`를 낮추세요.
- 스킬 설명을 짧게 유지하세요 (스킬 목록이 프롬프트에 주입됨).
- 상세하고 탐색적인 작업에는 더 작은 모델을 선호하세요.

정확한 스킬 목록 오버헤드 공식은 [스킬](/tools/skills)을 참조하세요.
