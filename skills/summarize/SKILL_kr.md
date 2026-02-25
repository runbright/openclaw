---
name: summarize
description: URL, 팟캐스트, 로컬 파일에서 텍스트나 트랜스크립트를 추출하여 요약합니다. 유튜브 영상의 내용을 파악하거나 자막을 추출할 때 매우 유용합니다.
homepage: https://summarize.sh
metadata:
  {
    "openclaw":
      {
        "emoji": "🧾",
        "requires": { "bins": ["summarize"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/summarize",
              "bins": ["summarize"],
              "label": "summarize 설치 (brew)",
            },
          ],
      },
  }
---

# 요약(Summarize) 스킬

URL, 로컬 파일, 유튜브 링크를 빠르게 요약해주는 CLI 도구입니다.

## 사용 사례 (트리거 문구)
사용자가 다음과 같은 요청을 할 때 즉시 이 스킬을 사용하세요:
- "이 링크/영상 내용이 뭐야?"
- "이 뉴스 기사 요약해줘"
- "이 유튜브 영상 자막(스크립트) 뽑아줘"
- "파일 내용을 간단하게 정리해줘"

## 명령어 예시
```bash
# 웹사이트 요약
summarize "https://example.com"

# 로컬 PDF 요약
summarize "/path/to/file.pdf"

# 유튜브 영상 요약
summarize "https://youtu.be/..." --youtube auto
```

## 유용한 옵션
- `--length short|medium|long`: 요약 길이 설정
- `--extract-only`: 요약하지 않고 전체 텍스트만 추출 (URL 전용)
- `--json`: 기계 읽기 가능한 형식으로 출력

---
상세한 모델 설정이나 추가 옵션은 영문 문서를 참조하세요.
---
