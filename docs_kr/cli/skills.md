---
summary: "`openclaw skills` (목록/정보/검사) 및 스킬 적격성 CLI 레퍼런스"
read_when:
  - 사용 가능하고 실행 준비된 스킬을 확인하고 싶을 때
  - 스킬에 누락된 바이너리/환경/구성을 디버깅하고 싶을 때
title: "skills"
---

# `openclaw skills`

스킬(번들 + 워크스페이스 + 관리 재정의)을 검사하고 적격한 것과 요구 사항이 누락된 것을 확인합니다.

관련 항목:

- 스킬 시스템: [스킬](/tools/skills)
- 스킬 구성: [스킬 구성](/tools/skills-config)
- ClawHub 설치: [ClawHub](/tools/clawhub)

## 명령어

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
