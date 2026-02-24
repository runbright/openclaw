---
summary: Node + tsx "__name is not a function" 충돌 노트 및 해결 방법
read_when:
  - Debugging Node-only dev scripts or watch mode failures
  - Investigating tsx/esbuild loader crashes in OpenClaw
title: "Node + tsx 충돌"
---

# Node + tsx "\_\_name is not a function" 충돌

## 요약

Node에서 `tsx`를 사용하여 OpenClaw를 실행하면 시작 시 다음 오류와 함께 실패합니다:

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

이 문제는 개발 스크립트를 Bun에서 `tsx`로 전환한 이후 시작되었습니다(커밋 `2871657e`, 2026-01-06). 동일한 런타임 경로가 Bun에서는 정상 작동했습니다.

## 환경

- Node: v25.x (v25.3.0에서 확인됨)
- tsx: 4.21.0
- OS: macOS (다른 Node 25 실행 플랫폼에서도 재현 가능할 것으로 보임)

## 재현 (Node 전용)

```bash
# 저장소 루트에서
node --version
pnpm install
node --import tsx src/entry.ts status
```

## 저장소 내 최소 재현

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```

## Node 버전 확인

- Node 25.3.0: 실패
- Node 22.22.0 (Homebrew `node@22`): 실패
- Node 24: 아직 설치되지 않음; 확인 필요

## 노트 / 가설

- `tsx`는 esbuild를 사용하여 TS/ESM을 변환합니다. esbuild의 `keepNames`는 `__name` 헬퍼를 생성하고 함수 정의를 `__name(...)`으로 래핑합니다.
- 충돌은 런타임에서 `__name`이 존재하지만 함수가 아님을 나타내며, 이는 Node 25 로더 경로에서 이 모듈에 대해 헬퍼가 누락되었거나 덮어씌워졌음을 의미합니다.
- 유사한 `__name` 헬퍼 문제가 헬퍼가 누락되거나 재작성된 다른 esbuild 소비자에서 보고되었습니다.

## 회귀 이력

- `2871657e` (2026-01-06): 스크립트가 Bun에서 tsx로 변경되어 Bun을 선택 사항으로 만듦.
- 그 이전(Bun 경로), `openclaw status`와 `gateway:watch`가 작동했습니다.

## 해결 방법

- 개발 스크립트에 Bun 사용 (현재 임시 되돌리기).
- Node + tsc watch를 사용한 다음, 컴파일된 출력 실행:

  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```

- 로컬에서 확인됨: `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status`가 Node 25에서 작동합니다.
- 가능한 경우 TS 로더에서 esbuild keepNames 비활성화(`__name` 헬퍼 삽입 방지); tsx는 현재 이를 노출하지 않습니다.
- Node LTS (22/24)에서 `tsx`로 테스트하여 문제가 Node 25에 한정되는지 확인합니다.

## 참고 자료

- [https://opennext.js.org/cloudflare/howtos/keep_names](https://opennext.js.org/cloudflare/howtos/keep_names)
- [https://esbuild.github.io/api/#keep-names](https://esbuild.github.io/api/#keep-names)
- [https://github.com/evanw/esbuild/issues/1031](https://github.com/evanw/esbuild/issues/1031)

## 다음 단계

- Node 22/24에서 재현하여 Node 25 회귀를 확인합니다.
- `tsx` 나이틀리를 테스트하거나 알려진 회귀가 있는 경우 이전 버전으로 고정합니다.
- Node LTS에서도 재현되면, `__name` 스택 추적과 함께 최소 재현을 업스트림에 제출합니다.
