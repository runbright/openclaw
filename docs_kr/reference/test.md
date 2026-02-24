---
summary: "로컬에서 테스트 실행 방법 (vitest) 및 force/coverage 모드 사용 시점"
read_when:
  - Running or fixing tests
title: "테스트"
---

# 테스트

- 전체 테스트 킷 (스위트, 라이브, Docker): [테스팅](/help/testing)

- `pnpm test:force`: 기본 컨트롤 포트를 점유하고 있는 게이트웨이 프로세스를 종료한 후, 격리된 게이트웨이 포트로 전체 Vitest 스위트를 실행하여 서버 테스트가 실행 중인 인스턴스와 충돌하지 않도록 합니다. 이전 게이트웨이 실행이 포트 18789를 점유한 상태로 남아 있을 때 사용하세요.
- `pnpm test:coverage`: V8 커버리지로 유닛 스위트를 실행합니다 (`vitest.unit.config.ts` 경유). 전역 임계값은 70% lines/branches/functions/statements입니다. 커버리지는 유닛 테스트 가능한 로직에 집중하기 위해 통합 중심 엔트리포인트(CLI 배선, 게이트웨이/텔레그램 브리지, 웹챗 정적 서버)를 제외합니다.
- Node 24+에서의 `pnpm test`: OpenClaw은 자동으로 Vitest `vmForks`를 비활성화하고 `forks`를 사용하여 `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`를 방지합니다. `OPENCLAW_TEST_VM_FORKS=0|1`로 동작을 강제할 수 있습니다.
- `pnpm test:e2e`: 게이트웨이 엔드투엔드 스모크 테스트를 실행합니다 (멀티 인스턴스 WS/HTTP/node 페어링). `vitest.e2e.config.ts`에서 기본값 `vmForks` + 적응형 워커; `OPENCLAW_E2E_WORKERS=<n>`으로 튜닝하고 상세 로그를 위해 `OPENCLAW_E2E_VERBOSE=1` 설정.
- `pnpm test:live`: 프로바이더 라이브 테스트를 실행합니다 (minimax/zai). API 키와 `LIVE=1` (또는 프로바이더별 `*_LIVE_TEST=1`)이 필요합니다.

## 모델 지연시간 벤치마크 (로컬 키)

스크립트: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

사용법:

- `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
- 선택적 환경변수: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
- 기본 프롬프트: "Reply with a single word: ok. No punctuation or extra text."

마지막 실행 (2025-12-31, 20회):

- minimax 중앙값 1279ms (최소 1114, 최대 2431)
- opus 중앙값 2454ms (최소 1224, 최대 3170)

## 온보딩 E2E (Docker)

Docker는 선택 사항입니다; 컨테이너화된 온보딩 스모크 테스트에만 필요합니다.

클린 Linux 컨테이너에서의 전체 콜드 스타트 플로우:

```bash
scripts/e2e/onboard-docker.sh
```

이 스크립트는 의사 tty를 통해 인터랙티브 위자드를 구동하고, 설정/워크스페이스/세션 파일을 검증한 후, 게이트웨이를 시작하고 `openclaw health`를 실행합니다.

## QR 임포트 스모크 (Docker)

Docker에서 Node 22+에서 `qrcode-terminal`이 로드되는지 확인:

```bash
pnpm test:docker:qr
```
