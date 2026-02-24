---
summary: "레퍼런스: 프로바이더별 트랜스크립트 위생처리 및 복구 규칙"
read_when:
  - You are debugging provider request rejections tied to transcript shape
  - You are changing transcript sanitization or tool-call repair logic
  - You are investigating tool-call id mismatches across providers
title: "트랜스크립트 위생"
---

# 트랜스크립트 위생 (프로바이더 수정)

이 문서는 실행 전(모델 컨텍스트 구성 시) 트랜스크립트에 적용되는 **프로바이더별 수정**을 설명합니다.
이것은 엄격한 프로바이더 요구사항을 충족하기 위해 사용되는 **인메모리** 조정입니다.
이 위생 단계는 디스크의 저장된 JSONL 트랜스크립트를 재작성하지 **않습니다**;
다만, 별도의 세션 파일 복구 패스가 세션 로드 전에 잘못된 JSONL 라인을 삭제하여 잘못된 형식의 JSONL 파일을 재작성할 수 있습니다.
복구가 발생하면 원본 파일이 세션 파일 옆에 백업됩니다.

범위:

- 도구 호출 id 위생처리
- 도구 호출 입력 검증
- 도구 결과 페어링 복구
- 턴 검증 / 순서 정렬
- 사고 서명 정리
- 이미지 페이로드 위생처리
- 사용자 입력 출처 태깅 (세션 간 라우팅된 프롬프트용)

트랜스크립트 저장 세부사항은 다음을 참조하세요:

- [/reference/session-management-compaction](/reference/session-management-compaction)

---

## 실행 위치

모든 트랜스크립트 위생은 임베디드 러너에 집중됩니다:

- 정책 선택: `src/agents/transcript-policy.ts`
- 위생처리/복구 적용: `src/agents/pi-embedded-runner/google.ts`의 `sanitizeSessionHistory`

정책은 `provider`, `modelApi`, `modelId`를 사용하여 무엇을 적용할지 결정합니다.

트랜스크립트 위생과 별도로, 세션 파일은 로드 전에 (필요시) 복구됩니다:

- `src/agents/session-file-repair.ts`의 `repairSessionFileIfNeeded`
- `run/attempt.ts` 및 `compact.ts` (임베디드 러너)에서 호출

---

## 글로벌 규칙: 이미지 위생처리

이미지 페이로드는 항상 위생처리되어 크기 제한으로 인한 프로바이더 측 거부를 방지합니다
(과대 base64 이미지를 축소/재압축).

이것은 비전 지원 모델의 이미지 기반 토큰 압력 제어에도 도움됩니다.
최대 크기를 낮추면 일반적으로 토큰 사용량이 줄고; 높이면 디테일이 보존됩니다.

구현:

- `src/agents/pi-embedded-helpers/images.ts`의 `sanitizeSessionMessagesImages`
- `src/agents/tool-images.ts`의 `sanitizeContentBlocksImages`
- 최대 이미지 변은 `agents.defaults.imageMaxDimensionPx` (기본값: `1200`)로 설정 가능.

---

## 글로벌 규칙: 잘못된 형식의 도구 호출

`input`과 `arguments`가 모두 없는 어시스턴트 도구 호출 블록은
모델 컨텍스트 구성 전에 삭제됩니다. 이것은 부분적으로 영속된 도구 호출
(예: 속도 제한 실패 후)로 인한 프로바이더 거부를 방지합니다.

구현:

- `src/agents/session-transcript-repair.ts`의 `sanitizeToolCallInputs`
- `src/agents/pi-embedded-runner/google.ts`의 `sanitizeSessionHistory`에서 적용

---

## 글로벌 규칙: 세션 간 입력 출처

에이전트가 `sessions_send`를 통해 다른 세션에 프롬프트를 보낼 때
(에이전트 간 reply/announce 단계 포함), OpenClaw은 생성된 사용자 턴을 다음과 함께 영속합니다:

- `message.provenance.kind = "inter_session"`

이 메타데이터는 트랜스크립트 추가 시점에 기록되며 역할을 변경하지 않습니다
(프로바이더 호환성을 위해 `role: "user"`가 유지됨). 트랜스크립트 리더는
라우팅된 내부 프롬프트를 최종 사용자가 작성한 지침으로 취급하지 않기 위해 이를 사용할 수 있습니다.

컨텍스트 재구성 중, OpenClaw은 모델이 외부 최종 사용자 지침과
구별할 수 있도록 해당 사용자 턴 앞에 짧은 `[Inter-session message]`
마커를 인메모리로 추가합니다.

---

## 프로바이더 매트릭스 (현재 동작)

**OpenAI / OpenAI Codex**

- 이미지 위생처리만.
- OpenAI Responses/Codex 트랜스크립트의 고아 추론 서명 삭제 (뒤따르는 콘텐츠 블록 없는 단독 추론 항목).
- 도구 호출 id 위생처리 없음.
- 도구 결과 페어링 복구 없음.
- 턴 검증 또는 재정렬 없음.
- 합성 도구 결과 없음.
- 사고 서명 제거 없음.

**Google (Generative AI / Gemini CLI / Antigravity)**

- 도구 호출 id 위생처리: 엄격한 영숫자.
- 도구 결과 페어링 복구 및 합성 도구 결과.
- 턴 검증 (Gemini 스타일 턴 교대).
- Google 턴 순서 수정 (히스토리가 어시스턴트로 시작하면 작은 사용자 부트스트랩 추가).
- Antigravity Claude: 사고 서명 정규화; 서명되지 않은 사고 블록 삭제.

**Anthropic / Minimax (Anthropic 호환)**

- 도구 결과 페어링 복구 및 합성 도구 결과.
- 턴 검증 (엄격한 교대를 만족하기 위해 연속 사용자 턴 병합).

**Mistral (model-id 기반 감지 포함)**

- 도구 호출 id 위생처리: strict9 (영숫자 길이 9).

**OpenRouter Gemini**

- 사고 서명 정리: non-base64 `thought_signature` 값 제거 (base64 유지).

**기타**

- 이미지 위생처리만.

---

## 역사적 동작 (2026.1.22 이전)

2026.1.22 릴리스 이전, OpenClaw은 여러 계층의 트랜스크립트 위생을 적용했습니다:

- **트랜스크립트 위생처리 확장**이 모든 컨텍스트 구성에서 실행되었으며 다음을 수행할 수 있었습니다:
  - 도구 사용/결과 페어링 복구.
  - 도구 호출 id 위생처리 (`_`/`-`를 보존하는 비엄격 모드 포함).
- 러너도 프로바이더별 위생처리를 수행하여 작업이 중복되었습니다.
- 프로바이더 정책 외부에서 추가 변형이 발생했으며, 다음을 포함:
  - 영속 전 어시스턴트 텍스트에서 `<final>` 태그 제거.
  - 빈 어시스턴트 에러 턴 삭제.
  - 도구 호출 후 어시스턴트 콘텐츠 자르기.

이 복잡성은 프로바이더 간 회귀를 유발했습니다 (특히 `openai-responses`
`call_id|fc_id` 페어링). 2026.1.22 정리에서 확장을 제거하고, 러너에 로직을 집중시키고,
OpenAI를 이미지 위생처리 외에 **변경 없음**으로 만들었습니다.
