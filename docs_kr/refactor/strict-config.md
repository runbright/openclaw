---
summary: "엄격한 구성 검증 + doctor 전용 마이그레이션"
read_when:
  - Designing or implementing config validation behavior
  - Working on config migrations or doctor workflows
  - Handling plugin config schemas or plugin load gating
title: "엄격한 구성 검증"
---

# 엄격한 구성 검증 (doctor 전용 마이그레이션)

## 목표

- 루트 `$schema` 메타데이터를 제외하고 **어디서든 알 수 없는 구성 키를 거부** (루트 + 중첩).
- **스키마 없는 플러그인 구성을 거부**; 해당 플러그인을 로드하지 않음.
- **로드 시 레거시 자동 마이그레이션 제거**; 마이그레이션은 doctor를 통해서만 실행.
- **시작 시 doctor(dry-run) 자동 실행**; 유효하지 않으면 비진단 명령을 차단.

## 비목표

- 로드 시 하위 호환성 (레거시 키가 자동 마이그레이션되지 않음).
- 인식되지 않는 키의 자동 삭제.

## 엄격한 검증 규칙

- 구성은 모든 레벨에서 스키마와 정확히 일치해야 함.
- 알 수 없는 키는 검증 오류 (루트 또는 중첩에서 통과 없음), 단 루트 `$schema`가 문자열인 경우 제외.
- `plugins.entries.<id>.config`는 플러그인의 스키마로 검증되어야 함.
  - 플러그인에 스키마가 없으면 **플러그인 로드를 거부**하고 명확한 오류를 표시.
- 플러그인 매니페스트가 채널 id를 선언하지 않는 한 알 수 없는 `channels.<id>` 키는 오류.
- 모든 플러그인에 플러그인 매니페스트 (`openclaw.plugin.json`)가 필수.

## 플러그인 스키마 시행

- 각 플러그인은 구성에 대한 엄격한 JSON Schema를 제공 (매니페스트에 인라인).
- 플러그인 로드 흐름:
  1. 플러그인 매니페스트 + 스키마 해결 (`openclaw.plugin.json`).
  2. 스키마에 대해 구성 검증.
  3. 스키마가 누락되거나 구성이 유효하지 않으면: 플러그인 로드 차단, 오류 기록.
- 오류 메시지 포함:
  - 플러그인 id
  - 이유 (스키마 누락 / 유효하지 않은 구성)
  - 검증에 실패한 경로
- 비활성화된 플러그인은 구성을 유지하지만, Doctor + 로그에 경고를 표시.

## Doctor 흐름

- 구성이 로드될 때 **매번** Doctor 실행 (기본적으로 dry-run).
- 구성이 유효하지 않으면:
  - 요약 + 실행 가능한 오류 출력.
  - 지시: `openclaw doctor --fix`.
- `openclaw doctor --fix`:
  - 마이그레이션 적용.
  - 알 수 없는 키 제거.
  - 업데이트된 구성 작성.

## 명령 게이팅 (구성이 유효하지 않을 때)

허용 (진단 전용):

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

그 외 모든 것은 다음과 함께 하드 실패: "Config invalid. Run `openclaw doctor --fix`."

## 오류 UX 형식

- 단일 요약 헤더.
- 그룹화된 섹션:
  - 알 수 없는 키 (전체 경로)
  - 레거시 키 / 필요한 마이그레이션
  - 플러그인 로드 실패 (플러그인 id + 이유 + 경로)

## 구현 접점

- `src/config/zod-schema.ts`: 루트 passthrough 제거; 모든 곳에서 strict 객체.
- `src/config/zod-schema.providers.ts`: strict 채널 스키마 확인.
- `src/config/validation.ts`: 알 수 없는 키에서 실패; 레거시 마이그레이션 적용하지 않음.
- `src/config/io.ts`: 레거시 자동 마이그레이션 제거; 항상 doctor dry-run 실행.
- `src/config/legacy*.ts`: 사용을 doctor로만 이동.
- `src/plugins/*`: 스키마 레지스트리 + 게이팅 추가.
- `src/cli`의 CLI 명령 게이팅.

## 테스트

- 알 수 없는 키 거부 (루트 + 중첩).
- 플러그인 스키마 누락 -> 명확한 오류와 함께 플러그인 로드 차단.
- 유효하지 않은 구성 -> 진단 명령을 제외한 gateway 시작 차단.
- Doctor dry-run 자동; `doctor --fix`가 수정된 구성 작성.
