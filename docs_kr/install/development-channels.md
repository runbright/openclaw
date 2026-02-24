---
summary: "안정, 베타, 개발 채널: 의미, 전환, 태깅"
read_when:
  - stable/beta/dev 간 전환을 원할 때
  - 프리릴리스를 태깅하거나 게시할 때
title: "개발 채널"
---

# 개발 채널

최종 업데이트: 2026-01-21

OpenClaw은 세 가지 업데이트 채널을 제공합니다:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (테스트 중인 빌드).
- **dev**: `main`의 최신 HEAD (git). npm dist-tag: `dev` (게시 시).

빌드를 **beta**에 배포하고, 테스트한 후, **검증된 빌드를 `latest`로 승격**합니다.
버전 번호를 변경하지 않습니다 — dist-tag가 npm 설치의 기준입니다.

## 채널 전환

Git 체크아웃:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta`는 최신 일치하는 태그를 체크아웃합니다 (종종 같은 태그).
- `dev`는 `main`으로 전환하고 업스트림에 리베이스합니다.

npm/pnpm 전역 설치:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

이 명령은 해당하는 npm dist-tag (`latest`, `beta`, `dev`)를 통해 업데이트합니다.

`--channel`로 **명시적으로** 채널을 전환하면, OpenClaw은 설치 방법도 함께 맞춥니다:

- `dev`는 git 체크아웃을 확보하고 (기본값 `~/openclaw`, `OPENCLAW_GIT_DIR`로 재정의 가능), 업데이트한 후, 해당 체크아웃에서 전역 CLI를 설치합니다.
- `stable`/`beta`는 일치하는 dist-tag를 사용하여 npm에서 설치합니다.

팁: stable + dev를 병렬로 사용하려면 두 개의 클론을 유지하고 게이트웨이를 stable 클론에 연결하세요.

## 플러그인과 채널

`openclaw update`로 채널을 전환하면, OpenClaw은 플러그인 소스도 동기화합니다:

- `dev`는 git 체크아웃의 번들 플러그인을 우선합니다.
- `stable`과 `beta`는 npm으로 설치된 플러그인 패키지를 복원합니다.

## 태깅 모범 사례

- git 체크아웃이 도달할 릴리스에 태그를 지정하세요 (stable은 `vYYYY.M.D`, beta는 `vYYYY.M.D-beta.N`).
- `vYYYY.M.D.beta.N`도 호환성을 위해 인식되지만, `-beta.N`을 권장합니다.
- 레거시 `vYYYY.M.D-<patch>` 태그는 여전히 stable (비 beta)로 인식됩니다.
- 태그를 불변으로 유지하세요: 태그를 이동하거나 재사용하지 마세요.
- npm dist-tag가 npm 설치의 기준으로 유지됩니다:
  - `latest` → stable
  - `beta` → 후보 빌드
  - `dev` → main 스냅샷 (선택 사항)

## macOS 앱 가용성

베타 및 개발 빌드에는 macOS 앱 릴리스가 포함되지 **않을** 수 있습니다. 이것은 정상입니다:

- git 태그와 npm dist-tag는 여전히 게시할 수 있습니다.
- 릴리스 노트나 변경 로그에 "이 베타에는 macOS 빌드 없음"을 명시하세요.
