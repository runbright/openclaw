---
title: CI 파이프라인
description: OpenClaw CI 파이프라인 작동 방식 안내
summary: "CI 작업 그래프, 스코프 게이트 및 로컬 명령어 안내"
read_when:
  - CI 작업이 실행되었거나 실행되지 않은 이유를 알고 싶을 때
  - GitHub Actions 체크 실패를 디버깅할 때
---

# CI 파이프라인 (CI Pipeline)

OpenClaw의 CI(지속적 통합)는 `main` 브랜치로의 모든 푸시와 모든 풀 리퀘스트(PR)에 대해 실행됩니다. 스마트 스코핑 기능을 사용하여 문서 전용 수정이나 네이티브 코드 변경 시 불필요하게 고비용 작업을 실행하지 않도록 최적화되어 있습니다.

## 작업 개요 (Job Overview)

| 작업 | 목적 | 실행 시점 |
| :--- | :--- | :--- |
| `docs-scope` | 문서 전용 변경 사항 감지 | 항상 |
| `changed-scope` | 변경 부위 감지 (node/macos/android) | 문서 외 PR 시 |
| `check` | TypeScript 타입, 린트, 포맷 체크 | 문서 외 변경 시 |
| `check-docs` | 마크다운 린트 + 깨진 링크 검사 | 문서 변경 시 |
| `code-analysis` | 코드 라인 수 임계치 체크 (1000라인) | PR만 해당 |
| `secrets` | 유출된 비밀 정보(API 키 등) 감지 | 항상 |
| `build-artifacts` | 빌드 수행 및 결과물 공유 | 문서 외, node 변경 시 |
| `release-check` | npm 패키지 내용 검증 | 빌드 후 |
| `checks` | Node/Bun 테스트 + 프로토콜 체크 | 문서 외, node 변경 시 |
| `checks-windows` | Windows 전용 테스트 | 문서 외, node 변경 시 |
| `macos` | Swift 린트/빌드/테스트 + TS 테스트 | macOS 변경 사항 포함 시 |
| `android` | Gradle 빌드 + 테스트 | 문서 외, Android 변경 시 |

## 로컬 실행 명령어 (Local Equivalents)

CI를 기다리지 않고 로컬에서 직접 확인하려면 다음 명령어를 사용하세요:

```bash
pnpm check          # 타입 + 린트 + 포맷 체크
pnpm test           # Vitest 테스트 실행
pnpm check:docs     # 문서 포맷 + 린트 + 링크 검사
pnpm release:check  # npm 패키지 구성 확인
```
---
