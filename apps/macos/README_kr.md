# OpenClaw macOS 앱 (개발 및 서명)

## 빠른 개발 실행
```bash
# 저장소 루트에서 실행
scripts/restart-mac.sh
```

**옵션:**
- `scripts/restart-mac.sh --no-sign`: 가장 빠른 개발 모드; 임시(Ad-hoc) 서명을 사용하며 TCC 권한이 유지되지 않을 수 있습니다.
- `scripts/restart-mac.sh --sign`: 코드 서명을 강제합니다 (인증서 필요).

## 패키징 프로세스
```bash
scripts/package-mac-app.sh
```
이 스트립트는 `dist/OpenClaw.app`을 생성하고 `scripts/codesign-mac-app.sh`를 통해 서명합니다.

## 코드 서명 규칙
자동으로 다음 순서에 따라 인증서를 선택합니다:
1) Developer ID Application
2) Apple Distribution
3) Apple Development
4) 사용 가능한 첫 번째 인증서

인증서를 찾을 수 없는 경우 기본적으로 오류가 발생합니다. 임시 서명을 하려면 `ALLOW_ADHOC_SIGNING=1` 또는 `SIGN_IDENTITY="-"`를 설정하세요.

## 팀 ID 감사 (Sparkle 미스매치 방지)
서명 후 앱 번들의 팀 ID를 읽어 앱 내부의 모든 Mach-O 바이너리와 비교합니다. 팀 ID가 다른 바이너리가 포함된 경우 서명이 실패합니다.
이 감사를 건너뛰려면 `SKIP_TEAM_ID_CHECK=1`을 설정하세요.

## 유용한 환경 변수 플래그
- `SIGN_IDENTITY="인증서 명칭"`: 특정 서명 ID 지정
- `ALLOW_ADHOC_SIGNING=1`: 임시 서명 허용
- `DISABLE_LIBRARY_VALIDATION=1`: (개발용) 라이브러리 유효성 검사 비활성화

---
상세한 릴리스 가이드는 `docs/platforms/mac/release.md`를 참조하세요.
---
