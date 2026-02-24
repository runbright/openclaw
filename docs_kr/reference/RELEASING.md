---
title: "릴리스 체크리스트"
summary: "npm + macOS 앱의 단계별 릴리스 체크리스트"
read_when:
  - Cutting a new npm release
  - Cutting a new macOS app release
  - Verifying metadata before publishing
---

# 릴리스 체크리스트 (npm + macOS)

저장소 루트에서 `pnpm` (Node 22+)을 사용하세요. 태그/퍼블리시 전에 작업 트리를 깨끗하게 유지하세요.

## 운영자 트리거

운영자가 "release"라고 하면, 즉시 이 사전 점검을 수행하세요 (차단되지 않는 한 추가 질문 없이):

- 이 문서와 `docs/platforms/mac/release.md`를 읽으세요.
- `~/.profile`에서 환경변수를 로드하고 `SPARKLE_PRIVATE_KEY_FILE` + App Store Connect 변수가 설정되어 있는지 확인하세요 (SPARKLE_PRIVATE_KEY_FILE은 `~/.profile`에 있어야 합니다).
- 필요하면 `~/Library/CloudStorage/Dropbox/Backup/Sparkle`의 Sparkle 키를 사용하세요.

1. **버전 및 메타데이터**

- [ ] `package.json` 버전 올리기 (예: `2026.1.29`).
- [ ] `pnpm plugins:sync`를 실행하여 확장 패키지 버전 + 변경 로그를 정렬.
- [ ] [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts)의 CLI/버전 문자열과 [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts)의 Baileys user agent를 업데이트.
- [ ] 패키지 메타데이터 (name, description, repository, keywords, license)와 `bin` 맵이 `openclaw`에 대해 [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)를 가리키는지 확인.
- [ ] 의존성이 변경되었으면 `pnpm install`을 실행하여 `pnpm-lock.yaml`을 최신 상태로 유지.

2. **빌드 및 아티팩트**

- [ ] A2UI 입력이 변경되었으면 `pnpm canvas:a2ui:bundle`을 실행하고 업데이트된 [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js)를 커밋.
- [ ] `pnpm run build` (`dist/` 재생성).
- [ ] npm 패키지 `files`에 필요한 모든 `dist/*` 폴더가 포함되어 있는지 확인 (특히 헤드리스 node + ACP CLI용 `dist/node-host/**` 및 `dist/acp/**`).
- [ ] `dist/build-info.json`이 존재하고 예상 `commit` 해시가 포함되어 있는지 확인 (CLI 배너가 npm 설치에 이를 사용).
- [ ] 선택 사항: 빌드 후 `npm pack --pack-destination /tmp`; tarball 내용을 검사하고 GitHub 릴리스용으로 준비 (커밋하지 **마세요**).

3. **변경 로그 및 문서**

- [ ] `CHANGELOG.md`를 사용자 대상 하이라이트로 업데이트 (파일이 없으면 생성); 항목은 버전별 내림차순 유지.
- [ ] README 예제/플래그가 현재 CLI 동작과 일치하는지 확인 (특히 새 명령어나 옵션).

4. **검증**

- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test` (또는 커버리지 출력이 필요하면 `pnpm test:coverage`)
- [ ] `pnpm release:check` (npm pack 내용 검증)
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (Docker 설치 스모크 테스트, 빠른 경로; 릴리스 전 필수)
  - 직전 npm 릴리스가 깨진 것으로 알려진 경우, 사전 설치 단계에 `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` 또는 `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1`을 설정.
- [ ] (선택 사항) 전체 설치 스모크 (non-root + CLI 커버리지 추가): `pnpm test:install:smoke`
- [ ] (선택 사항) 설치 E2E (Docker, `curl -fsSL https://openclaw.ai/install.sh | bash` 실행, 온보딩, 실제 도구 호출 실행):
  - `pnpm test:install:e2e:openai` (`OPENAI_API_KEY` 필요)
  - `pnpm test:install:e2e:anthropic` (`ANTHROPIC_API_KEY` 필요)
  - `pnpm test:install:e2e` (양쪽 키 필요; 두 프로바이더 모두 실행)
- [ ] (선택 사항) 변경사항이 전송/수신 경로에 영향을 미치면 웹 게이트웨이 확인.

5. **macOS 앱 (Sparkle)**

- [ ] macOS 앱을 빌드 + 서명한 후 배포용 zip 생성.
- [ ] Sparkle appcast를 생성 ([`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)를 통한 HTML 노트)하고 `appcast.xml` 업데이트.
- [ ] 앱 zip (그리고 선택적 dSYM zip)을 GitHub 릴리스에 첨부할 준비.
- [ ] 정확한 명령어와 필요한 환경변수는 [macOS 릴리스](/platforms/mac/release)를 따르세요.
  - `APP_BUILD`는 숫자 + 단조 증가여야 합니다 (`-beta` 없음), Sparkle이 버전을 올바르게 비교하도록.
  - 공증 시 App Store Connect API 환경변수로 생성된 `openclaw-notary` 키체인 프로필을 사용하세요 ([macOS 릴리스](/platforms/mac/release) 참조).

6. **퍼블리시 (npm)**

- [ ] git 상태가 깨끗한지 확인; 필요시 커밋 및 push.
- [ ] 필요시 `npm login` (2FA 확인).
- [ ] `npm publish --access public` (프리릴리스에는 `--tag beta` 사용).
- [ ] 레지스트리 확인: `npm view openclaw version`, `npm view openclaw dist-tags`, 그리고 `npx -y openclaw@X.Y.Z --version` (또는 `--help`).

### 트러블슈팅 (2.0.0-beta2 릴리스 참고사항)

- **npm pack/publish가 멈추거나 거대한 tarball 생성**: `dist/OpenClaw.app`의 macOS 앱 번들 (및 릴리스 zip)이 패키지에 포함됩니다. `package.json` `files`를 통해 퍼블리시 내용을 화이트리스트하여 수정 (dist 하위 디렉토리, docs, skills 포함; 앱 번들 제외). `npm pack --dry-run`으로 `dist/OpenClaw.app`이 목록에 없는지 확인.
- **dist-tags용 npm 인증 웹 루프**: 레거시 인증을 사용하여 OTP 프롬프트 받기:
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
- **`npx` 검증이 `ECOMPROMISED: Lock compromised`로 실패**: 새 캐시로 재시도:
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
- **늦은 수정 후 태그 재지정 필요**: 태그를 강제 업데이트하고 push, GitHub 릴리스 자산이 여전히 일치하는지 확인:
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub 릴리스 + appcast**

- [ ] 태그 및 push: `git tag vX.Y.Z && git push origin vX.Y.Z` (또는 `git push --tags`).
- [ ] `vX.Y.Z`에 대한 GitHub 릴리스를 생성/갱신하고 **제목을 `openclaw X.Y.Z`**로 설정 (태그만이 아님); 본문에는 해당 버전의 **전체** 변경 로그 섹션 (Highlights + Changes + Fixes)을 인라인으로 포함하고 (링크만 나열하지 않음), **본문 내에 제목을 반복하지 마세요**.
- [ ] 아티팩트 첨부: `npm pack` tarball (선택), `OpenClaw-X.Y.Z.zip`, 그리고 `OpenClaw-X.Y.Z.dSYM.zip` (생성된 경우).
- [ ] 업데이트된 `appcast.xml`을 커밋하고 push (Sparkle는 main에서 피드).
- [ ] 깨끗한 임시 디렉토리에서 (`package.json` 없는 곳), `npx -y openclaw@X.Y.Z send --help`를 실행하여 설치/CLI 엔트리포인트가 동작하는지 확인.
- [ ] 릴리스 노트 공유/발표.

## 플러그인 퍼블리시 범위 (npm)

`@openclaw/*` 스코프 아래의 **기존 npm 플러그인**만 퍼블리시합니다. npm에 없는 번들
플러그인은 **디스크 트리로만** 유지됩니다 (`extensions/**`에 여전히 포함).

목록을 도출하는 프로세스:

1. `npm search @openclaw --json`으로 패키지 이름을 캡처.
2. `extensions/*/package.json` 이름과 비교.
3. **교집합** (이미 npm에 있는 것)만 퍼블리시.

현재 npm 플러그인 목록 (필요시 업데이트):

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

릴리스 노트에는 **기본적으로 활성화되지 않은** **새로운 선택적 번들 플러그인**도 명시해야 합니다 (예: `tlon`).
