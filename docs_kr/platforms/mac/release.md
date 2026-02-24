---
summary: "OpenClaw macOS 릴리스 체크리스트 (Sparkle 피드, 패키징, 서명)"
read_when:
  - OpenClaw macOS 릴리스 진행 또는 검증
  - Sparkle appcast 또는 피드 자산 업데이트
title: "macOS 릴리스"
---

# OpenClaw macOS 릴리스 (Sparkle)

이 앱은 이제 Sparkle 자동 업데이트를 제공합니다. 릴리스 빌드는 Developer ID 서명, zip 압축, 서명된 appcast 항목과 함께 게시되어야 합니다.

## 사전 요구사항

- Developer ID Application 인증서 설치됨 (예: `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Sparkle 비공개 키 경로가 환경 변수 `SPARKLE_PRIVATE_KEY_FILE`로 설정됨 (Sparkle ed25519 비공개 키 경로; 공개 키는 Info.plist에 내장). 없으면 `~/.profile`을 확인하세요.
- `xcrun notarytool`을 위한 공증 자격 증명 (Keychain 프로필 또는 API 키), Gatekeeper 안전한 DMG/zip 배포를 원하는 경우.
  - 쉘 프로필의 App Store Connect API 키 환경 변수로부터 생성된 `openclaw-notary`라는 이름의 Keychain 프로필을 사용합니다:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- `pnpm` 의존성 설치됨 (`pnpm install --config.node-linker=hoisted`).
- Sparkle 도구는 SwiftPM을 통해 자동으로 가져옵니다: `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast` 등).

## 빌드 및 패키징

참고:

- `APP_BUILD`는 `CFBundleVersion`/`sparkle:version`에 매핑됩니다; 숫자 + 단조 증가로 유지하세요 (`-beta` 없이), 그렇지 않으면 Sparkle이 동일하게 비교합니다.
- 기본값은 현재 아키텍처 (`$(uname -m)`)입니다. 릴리스/유니버설 빌드의 경우 `BUILD_ARCHS="arm64 x86_64"` (또는 `BUILD_ARCHS=all`)를 설정하세요.
- 릴리스 아티팩트 (zip + DMG + 공증)에는 `scripts/package-mac-dist.sh`를 사용하세요. 로컬/개발 패키징에는 `scripts/package-mac-app.sh`를 사용하세요.

```bash
# 저장소 루트에서; Sparkle 피드가 활성화되도록 릴리스 ID를 설정합니다.
# APP_BUILD는 Sparkle 비교를 위해 숫자 + 단조 증가여야 합니다.
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.2.23 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# 배포용 Zip (Sparkle 델타 지원을 위한 리소스 포크 포함)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.2.23.zip

# 선택사항: 사용자를 위한 스타일링된 DMG 빌드 (/Applications로 드래그)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.2.23.dmg

# 권장: zip + DMG 빌드 + 공증/스테이플
# 먼저 Keychain 프로필을 한 번 생성:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.2.23 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# 선택사항: 릴리스와 함께 dSYM 제공
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.2.23.dSYM.zip
```

## Appcast 항목

Sparkle이 포맷된 HTML 노트를 렌더링할 수 있도록 릴리스 노트 생성기를 사용하세요:

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.2.23.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

`CHANGELOG.md`에서 HTML 릴리스 노트를 생성하고 ([`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh) 사용) appcast 항목에 임베딩합니다.
게시 시 업데이트된 `appcast.xml`을 릴리스 자산 (zip + dSYM)과 함께 커밋하세요.

## 게시 및 확인

- `OpenClaw-2026.2.23.zip` (및 `OpenClaw-2026.2.23.dSYM.zip`)을 `v2026.2.23` 태그의 GitHub 릴리스에 업로드하세요.
- raw appcast URL이 내장 피드와 일치하는지 확인하세요: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- 온전성 검사:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`이 200을 반환하는지.
  - 자산 업로드 후 `curl -I <enclosure url>`이 200을 반환하는지.
  - 이전 공개 빌드에서 About 탭의 "Check for Updates..."를 실행하고 Sparkle이 새 빌드를 깨끗하게 설치하는지 확인.

완료 정의: 서명된 앱 + appcast가 게시되고, 이전 설치 버전에서 업데이트 흐름이 작동하며, 릴리스 자산이 GitHub 릴리스에 첨부됨.
