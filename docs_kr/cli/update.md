---
summary: "`openclaw update` (안전한 소스 업데이트 + 게이트웨이 자동 재시작) CLI 레퍼런스"
read_when:
  - 소스 체크아웃을 안전하게 업데이트하고 싶을 때
  - `--update` 단축 동작을 이해해야 할 때
title: "update"
---

# `openclaw update`

OpenClaw를 안전하게 업데이트하고 stable/beta/dev 채널 간에 전환합니다.

**npm/pnpm** (전역 설치, git 메타데이터 없음)으로 설치한 경우, 업데이트는 [업데이트](/install/updating)의 패키지 매니저 플로우를 통해 수행됩니다.

## 사용법

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## 옵션

- `--no-restart`: 성공적인 업데이트 후 Gateway 서비스 재시작을 건너뜁니다.
- `--channel <stable|beta|dev>`: 업데이트 채널을 설정합니다 (git + npm; 구성에 유지).
- `--tag <dist-tag|version>`: 이번 업데이트에만 npm dist-tag 또는 버전을 재정의합니다.
- `--dry-run`: 구성 작성, 설치, 플러그인 동기화, 재시작 없이 계획된 업데이트 작업(채널/태그/대상/재시작 플로우)을 미리 봅니다.
- `--json`: 머신 판독 가능한 `UpdateRunResult` JSON을 출력합니다.
- `--timeout <seconds>`: 단계별 타임아웃 (기본값 1200초).

참고: 다운그레이드는 이전 버전이 구성을 깨뜨릴 수 있으므로 확인이 필요합니다.

## `update status`

활성 업데이트 채널 + git 태그/브랜치/SHA (소스 체크아웃의 경우) 및 업데이트 가용성을 표시합니다.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

옵션:

- `--json`: 머신 판독 가능한 상태 JSON을 출력합니다.
- `--timeout <seconds>`: 검사 타임아웃 (기본값 3초).

## `update wizard`

업데이트 채널을 선택하고 업데이트 후 Gateway를 재시작할지 확인하는 인터랙티브 플로우입니다
(기본값은 재시작). git 체크아웃 없이 `dev`를 선택하면 체크아웃을 생성할 것을 제안합니다.

## 동작 방식

채널을 명시적으로 전환하면 (`--channel ...`), OpenClaw는 설치 방법도 맞춥니다:

- `dev` -> git 체크아웃을 보장합니다 (기본값: `~/openclaw`, `OPENCLAW_GIT_DIR`로 재정의),
  업데이트하고, 해당 체크아웃에서 전역 CLI를 설치합니다.
- `stable`/`beta` -> 일치하는 dist-tag를 사용하여 npm에서 설치합니다.

Gateway 코어 자동 업데이터(구성을 통해 활성화된 경우)는 동일한 업데이트 경로를 재사용합니다.

## Git 체크아웃 플로우

채널:

- `stable`: 최신 비베타 태그를 체크아웃한 후 빌드 + doctor를 실행합니다.
- `beta`: 최신 `-beta` 태그를 체크아웃한 후 빌드 + doctor를 실행합니다.
- `dev`: `main`을 체크아웃한 후 fetch + rebase를 실행합니다.

개요:

1. 클린 워크트리가 필요합니다 (커밋되지 않은 변경 없음).
2. 선택한 채널(태그 또는 브랜치)로 전환합니다.
3. 업스트림을 가져옵니다 (dev만 해당).
4. dev만 해당: 임시 워크트리에서 프리플라이트 lint + TypeScript 빌드를 수행합니다; 팁이 실패하면 최대 10개 커밋을 거슬러 올라가 가장 최신의 클린 빌드를 찾습니다.
5. 선택한 커밋으로 rebase합니다 (dev만 해당).
6. 의존성을 설치합니다 (pnpm 우선; npm 폴백).
7. 빌드 + Control UI를 빌드합니다.
8. 최종 "안전 업데이트" 검사로 `openclaw doctor`를 실행합니다.
9. 플러그인을 활성 채널에 동기화합니다 (dev는 번들 확장을 사용; stable/beta는 npm 사용) 및 npm으로 설치된 플러그인을 업데이트합니다.

## `--update` 단축키

`openclaw --update`는 `openclaw update`로 변환됩니다 (셸 및 런처 스크립트에 유용).

## 참고 항목

- `openclaw doctor` (git 체크아웃에서 먼저 업데이트 실행을 제안)
- [개발 채널](/install/development-channels)
- [업데이트](/install/updating)
- [CLI 레퍼런스](/cli)
