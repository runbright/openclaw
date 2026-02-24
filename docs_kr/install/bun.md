---
summary: "Bun 워크플로우 (실험적): pnpm 대비 설치 방법 및 주의사항"
read_when:
  - 가장 빠른 로컬 개발 루프 (bun + watch)를 원할 때
  - Bun 설치/패치/라이프사이클 스크립트 문제를 겪을 때
title: "Bun (실험적)"
---

# Bun (실험적)

목표: pnpm 워크플로우와 분기하지 않으면서 이 저장소를 **Bun**으로 실행 (선택 사항, WhatsApp/Telegram에는 권장하지 않음)

⚠️ **게이트웨이 런타임에는 권장하지 않음** (WhatsApp/Telegram 버그). 프로덕션에는 Node를 사용하세요.

## 상태

- Bun은 TypeScript를 직접 실행하기 위한 선택적 로컬 런타임입니다 (`bun run …`, `bun --watch …`).
- `pnpm`이 빌드의 기본이며 완전히 지원됩니다 (일부 문서 도구에서도 사용).
- Bun은 `pnpm-lock.yaml`을 사용할 수 없으며 무시합니다.

## 설치

기본:

```sh
bun install
```

참고: `bun.lock`/`bun.lockb`는 gitignore에 포함되어 있어 어떤 방식이든 저장소 변동이 없습니다. _락파일 쓰기 없이_ 설치하려면:

```sh
bun install --no-save
```

## 빌드 / 테스트 (Bun)

```sh
bun run build
bun run vitest run
```

## Bun 라이프사이클 스크립트 (기본적으로 차단됨)

Bun은 명시적으로 신뢰하지 않은 의존성 라이프사이클 스크립트를 차단할 수 있습니다 (`bun pm untrusted` / `bun pm trust`).
이 저장소에서 일반적으로 차단되는 스크립트는 필수가 아닙니다:

- `@whiskeysockets/baileys` `preinstall`: Node 메이저 버전 >= 20인지 확인 (우리는 Node 22+ 사용).
- `protobufjs` `postinstall`: 호환되지 않는 버전 체계에 대한 경고 출력 (빌드 아티팩트 없음).

이러한 스크립트가 필요한 실제 런타임 문제가 발생하면 명시적으로 신뢰하세요:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## 주의사항

- 일부 스크립트는 여전히 pnpm을 하드코딩합니다 (예: `docs:build`, `ui:*`, `protocol:check`). 현재는 pnpm으로 실행하세요.
