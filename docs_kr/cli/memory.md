---
summary: "`openclaw memory` (상태/인덱스/검색) CLI 레퍼런스"
read_when:
  - 시맨틱 메모리를 인덱싱하거나 검색하고 싶을 때
  - 메모리 가용성이나 인덱싱을 디버깅할 때
title: "memory"
---

# `openclaw memory`

시맨틱 메모리 인덱싱 및 검색을 관리합니다.
활성 메모리 플러그인에 의해 제공됩니다 (기본값: `memory-core`; 비활성화하려면 `plugins.slots.memory = "none"`으로 설정).

관련 항목:

- 메모리 개념: [메모리](/concepts/memory)
- 플러그인: [플러그인](/tools/plugin)

## 예시

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## 옵션

공통:

- `--agent <id>`: 단일 에이전트로 범위를 지정합니다 (기본값: 모든 구성된 에이전트).
- `--verbose`: 프로브 및 인덱싱 중 상세 로그를 출력합니다.

참고:

- `memory status --deep`은 벡터 + 임베딩 가용성을 프로브합니다.
- `memory status --deep --index`는 스토어가 변경된 경우 재인덱싱을 실행합니다.
- `memory index --verbose`는 단계별 상세 정보(프로바이더, 모델, 소스, 배치 활동)를 출력합니다.
- `memory status`는 `memorySearch.extraPaths`를 통해 구성된 추가 경로를 포함합니다.
