---
name: github
description: "`gh` CLI를 통한 GitHub 작업: 이슈, PR, CI 실행, 코드 리뷰, API 쿼리. 사용 사례: (1) PR 상태 또는 CI 확인, (2) 이슈 생성 및 댓글 달기, (3) PR/이슈 목록 필터링, (4) 실행 로그 확인. 제외 사례: 복잡한 브라우저 기반 UI 상호작용(브라우저 도구 사용), 대량의 저장소에 대한 벌크 작업(gh api 스크립트 사용), gh 인증이 설정되지 않은 경우."
metadata:
  {
    "openclaw":
      {
        "emoji": "🐙",
        "requires": { "bins": ["gh"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gh",
              "bins": ["gh"],
              "label": "GitHub CLI 설치 (brew)",
            },
            {
              "id": "apt",
              "kind": "apt",
              "package": "gh",
              "bins": ["gh"],
              "label": "GitHub CLI 설치 (apt)",
            },
          ],
      },
  }
---

# GitHub 스킬

`gh` CLI를 사용하여 GitHub 저장소, 이슈, PR 및 CI와 상호작용합니다.

## 사용 사례

✅ **다음과 같을 때 이 스킬을 사용하세요:**
- PR 상태, 리뷰 결과 또는 머지 준비 상태 확인
- CI/워크플로우 실행 상태 및 로그 확인
- 이슈 생성, 종료 또는 댓글 달기
- 풀 리퀘스트 생성 또는 머지
- 저장소 데이터에 대한 GitHub API 쿼리
- 저장소 목록, 릴리스 또는 협업자 조회

❌ **다음과 같을 때 이 스킬을 사용하지 마세요:**
- 로컬 git 작업 (commit, push, pull, branch) → 직접 `git` 명령어 사용
- GitHub 이외의 저장소 (GitLab, Bitbucket 등) → 해당 플랫폼의 CLI 사용
- 저장소 클론 → `git clone` 사용
- 실제 코드 변경 사항 리뷰 → `coding-agent` 스킬 사용

## 설정 방법
```bash
# 인증 (최초 1회)
gh auth login

# 확인
gh auth status
```

## 주요 명령어 예시
### 풀 리퀘스트 (PR)
```bash
# PR 목록 조회
gh pr list --repo owner/repo

# CI 상태 확인
gh pr checks 55 --repo owner/repo

# PR 상세 내용 보기
gh pr view 55 --repo owner/repo

# PR 머지
gh pr merge 55 --squash --repo owner/repo
```

---
상세한 사용법이나 추가 명령어는 영문 문서를 참조하세요.
---
