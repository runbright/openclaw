---
summary: "모델 관리 CLI 사용법: 목록 확인, 설정, 별칭(Alias), 대체 모델 관리 및 상태 점검"
read_when:
  - 모델 관련 CLI 명령어(list, set, scan 등)를 사용하거나 수정할 때
  - 대체 모델 동작 방식이나 모델 선택 UX를 변경할 때
title: "모델 관리 CLI"
---

# 모델 관리 CLI (Models CLI)

OpenClaw는 다양한 언어 모델(LLM)을 유연하게 전환하고 관리할 수 있는 강력한 명령줄 도구(CLI)를 제공합니다.

## 모델 선택 우선순위

OpenClaw는 메시지를 처리할 때 다음 순서대로 모델을 선택합니다:

1. **기본 모델 (Primary)**: `agents.defaults.model`에 설정된 주 모델입니다.
2. **대체 모델 (Fallbacks)**: 주 모델이 실패할 경우 시도할 모델들의 목록입니다.
3. **인증 프로필 전환**: 동일한 모델 내에서 여러 계정이 있다면 계정을 먼저 전환하며 시도합니다.

## 주요 CLI 명령어

### 1. 모델 상태 및 목록 확인
- `openclaw models status`: 현재 설정된 주 모델, 대체 모델 및 인증 계정의 상태(만료 여부 등)를 확인합니다.
- `openclaw models list`: 사용 가능한 모든 모델 목록을 보여줍니다. (별칭 포함)

### 2. 모델 설정 및 변경
- `openclaw models set <제공사/모델ID>`: 기본 모델을 변경합니다. (예: `openai/gpt-4o`)
- `openclaw models set-image <제공사/모델ID>`: 이미지 처리를 전담할 모델을 설정합니다.

### 3. 대체 모델(Fallbacks) 관리
- `openclaw models fallbacks add <제공사/모델ID>`: 새로운 대체 모델을 추가합니다.
- `openclaw models fallbacks list`: 현재 설정된 대체 모델 순서를 확인합니다.

### 4. 무료 모델 스캔 (`scan`)
- `openclaw models scan`: OpenRouter 등에서 제공하는 **무료 모델**들을 자동으로 찾아서 성능(도구 실행 능력, 이미지 인식 등)을 테스트하고 추천 목록을 만들어줍니다.

## 채팅 중에서 모델 바꾸기 (`/model`)

게이트웨이를 재시작하지 않고도 대화 도중에 모델을 바로 바꿀 수 있습니다:
- `/model`: 현재 사용 가능한 모델 목록을 번호와 함께 보여줍니다.
- `/model <번호>`: 해당 번호의 모델로 즉시 전환합니다.
- `/model status`: 현재 세션에서 사용 중인 모델과 인증 상태를 상세히 보여줍니다.

## "Model is not allowed" 오류 해결

설정 파일에 `models` 허용 목록(Allowlist)이 정의되어 있다면, 그 목록에 없는 모델은 사용할 수 없습니다. 이 오류가 발생하면 `openclaw.json`의 `models` 섹션에 해당 모델을 추가하거나, 허용 목록 설정을 비워두세요.

---
모델 제공사별 상세 설정은 [모델 제공사 가이드](/concepts/model-providers)를 참고하세요.
---
