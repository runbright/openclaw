---
summary: "/think + /verbose 지시문 구문 및 모델 추론에 미치는 영향"
read_when:
  - thinking 또는 verbose 지시문 파싱이나 기본값을 조정하는 경우
title: "Thinking 수준"
---

# Thinking 수준 (/think 지시문)

## 기능

- 인바운드 본문의 인라인 지시문: `/t <level>`, `/think:<level>` 또는 `/thinking <level>`.
- 수준(별칭): `off | minimal | low | medium | high | xhigh`(GPT-5.2 + Codex 모델 전용)
  - minimal → "think"
  - low → "think hard"
  - medium → "think harder"
  - high → "ultrathink"(최대 예산)
  - xhigh → "ultrathink+"(GPT-5.2 + Codex 모델 전용)
  - `x-high`, `x_high`, `extra-high`, `extra high`, `extra_high`는 `xhigh`로 매핑됩니다.
  - `highest`, `max`는 `high`로 매핑됩니다.
- 프로바이더 참고:
  - Z.AI(`zai/*`)는 바이너리 thinking(`on`/`off`)만 지원합니다. `off`가 아닌 모든 수준은 `on`(→ `low`로 매핑)으로 취급됩니다.

## 해석 순서

1. 메시지의 인라인 지시문(해당 메시지에만 적용).
2. 세션 재정의(지시문만 있는 메시지를 보내 설정).
3. 전역 기본값(구성의 `agents.defaults.thinkingDefault`).
4. 폴백: 추론 가능한 모델에는 low; 그렇지 않으면 off.

## 세션 기본값 설정

- **지시문만** 있는 메시지를 보냅니다(공백 허용), 예: `/think:medium` 또는 `/t high`.
- 현재 세션에 유지됩니다(기본적으로 발신자별); `/think:off`나 세션 유휴 리셋으로 해제됩니다.
- 확인 응답이 전송됩니다(`Thinking level set to high.` / `Thinking disabled.`). 수준이 유효하지 않으면(예: `/thinking big`) 명령은 힌트와 함께 거부되고 세션 상태는 변경되지 않습니다.
- 인수 없이 `/think`(또는 `/think:`)를 보내면 현재 thinking 수준을 확인할 수 있습니다.

## 에이전트별 적용

- **내장 Pi**: 해석된 수준이 인프로세스 Pi 에이전트 런타임에 전달됩니다.

## Verbose 지시문 (/verbose 또는 /v)

- 수준: `on`(최소) | `full` | `off`(기본값).
- 지시문만 있는 메시지는 세션 verbose를 토글하고 `Verbose logging enabled.` / `Verbose logging disabled.`로 응답합니다; 유효하지 않은 수준은 상태를 변경하지 않고 힌트를 반환합니다.
- `/verbose off`는 명시적 세션 재정의를 저장합니다; Sessions UI에서 `inherit`를 선택하여 해제하세요.
- 인라인 지시문은 해당 메시지에만 영향을 줍니다; 그렇지 않으면 세션/전역 기본값이 적용됩니다.
- 인수 없이 `/verbose`(또는 `/verbose:`)를 보내면 현재 verbose 수준을 확인할 수 있습니다.
- verbose가 켜져 있으면 구조화된 도구 결과를 발생시키는 에이전트(Pi, 기타 JSON 에이전트)가 각 도구 호출을 자체 메타데이터 전용 메시지로 보냅니다. 사용 가능한 경우 `<emoji> <tool-name>: <arg>` 접두사(경로/명령)가 붙습니다. 이 도구 요약은 각 도구가 시작될 때(별도 버블) 전송되며 스트리밍 델타가 아닙니다.
- 도구 실패 요약은 일반 모드에서도 표시되지만 원시 오류 세부 접미사는 verbose가 `on` 또는 `full`일 때만 표시됩니다.
- verbose가 `full`이면 도구 출력도 완료 후 전달됩니다(별도 버블, 안전한 길이로 잘림). 실행 중에 `/verbose on|full|off`를 토글하면 후속 도구 버블이 새 설정을 따릅니다.

## 추론 가시성 (/reasoning)

- 수준: `on|off|stream`.
- 지시문만 있는 메시지는 응답에 thinking 블록을 표시할지 여부를 토글합니다.
- 활성화되면 추론이 `Reasoning:` 접두사가 있는 **별도 메시지**로 전송됩니다.
- `stream`(Telegram 전용): 응답이 생성되는 동안 추론을 Telegram 초안 버블로 스트리밍한 다음 추론 없이 최종 답변을 전송합니다.
- 별칭: `/reason`.
- 인수 없이 `/reasoning`(또는 `/reasoning:`)을 보내면 현재 reasoning 수준을 확인할 수 있습니다.

## 관련

- Elevated 모드 문서는 [Elevated 모드](/tools/elevated)에 있습니다.

## 하트비트

- 하트비트 프로브 본문은 구성된 하트비트 프롬프트입니다(기본값: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). 하트비트 메시지의 인라인 지시문은 평소대로 적용됩니다(하트비트에서 세션 기본값을 변경하는 것은 피하세요).
- 하트비트 전달은 기본적으로 최종 페이로드만 전달합니다. 별도의 `Reasoning:` 메시지(사용 가능한 경우)도 전송하려면 `agents.defaults.heartbeat.includeReasoning: true` 또는 에이전트별 `agents.list[].heartbeat.includeReasoning: true`를 설정하세요.

## 웹 채팅 UI

- 웹 채팅 thinking 선택기는 페이지가 로드될 때 인바운드 세션 저장소/구성의 세션 저장 수준을 반영합니다.
- 다른 수준을 선택하면 다음 메시지에만 적용됩니다(`thinkingOnce`); 전송 후 선택기는 저장된 세션 수준으로 돌아갑니다.
- 세션 기본값을 변경하려면 이전과 같이 `/think:<level>` 지시문을 보내세요; 다음 새로 고침 후 선택기에 반영됩니다.
