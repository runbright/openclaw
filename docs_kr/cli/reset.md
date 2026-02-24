---
summary: "`openclaw reset` (로컬 상태/구성 초기화) CLI 레퍼런스"
read_when:
  - CLI를 설치된 상태로 유지하면서 로컬 상태를 삭제하고 싶을 때
  - 제거될 항목의 dry-run을 확인하고 싶을 때
title: "reset"
---

# `openclaw reset`

로컬 구성/상태를 초기화합니다 (CLI는 설치된 상태로 유지).

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
