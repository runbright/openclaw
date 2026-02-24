---
summary: "통합 브라우저 제어 서비스 + 액션 명령"
read_when:
  - 에이전트 제어 브라우저 자동화를 추가할 때
  - openclaw이 자신의 Chrome을 간섭하는 이유를 디버깅할 때
  - macOS 앱에서 브라우저 설정 + 생명주기를 구현할 때
title: "브라우저 (OpenClaw 관리형)"
---

# 브라우저 (openclaw 관리형)

OpenClaw는 에이전트가 제어하는 **전용 Chrome/Brave/Edge/Chromium 프로파일**을 실행할 수 있습니다.
개인 브라우저와 격리되어 있으며 게이트웨이 내부의 작은 로컬
제어 서비스를 통해 관리됩니다 (루프백 전용).

초보자 관점:

- **별도의, 에이전트 전용 브라우저**로 생각하세요.
- `openclaw` 프로파일은 개인 브라우저 프로파일을 **건드리지 않습니다**.
- 에이전트는 안전한 레인에서 **탭을 열고, 페이지를 읽고, 클릭하고, 입력**할 수 있습니다.
- 기본 `chrome` 프로파일은 확장 프로그램 릴레이를 통해 **시스템 기본 Chromium 브라우저**를 사용합니다; 격리된 관리형 브라우저를 사용하려면 `openclaw`로 전환하세요.

## 제공되는 기능

- **openclaw**이라는 별도의 브라우저 프로파일 (기본적으로 주황색 강조).
- 결정론적 탭 제어 (목록/열기/포커스/닫기).
- 에이전트 액션 (클릭/입력/드래그/선택), 스냅샷, 스크린샷, PDF.
- 선택적 다중 프로파일 지원 (`openclaw`, `work`, `remote`, ...).

이 브라우저는 일상적인 드라이버가 **아닙니다**. 에이전트 자동화 및 검증을 위한
안전하고 격리된 환경입니다.

## 빠른 시작

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

"Browser disabled"가 표시되면, 설정에서 활성화하고 (아래 참조) 게이트웨이를
재시작하세요.

## 프로파일: `openclaw` vs `chrome`

- `openclaw`: 관리형, 격리된 브라우저 (확장 프로그램 불필요).
- `chrome`: **시스템 브라우저**에 대한 확장 프로그램 릴레이 (OpenClaw 확장 프로그램이 탭에 연결되어야 함).

관리형 모드를 기본으로 사용하려면 `browser.defaultProfile: "openclaw"`를 설정하세요.

## 설정

브라우저 설정은 `~/.openclaw/openclaw.json`에 있습니다.

```json5
{
  browser: {
    enabled: true, // 기본값: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // 기본 신뢰 네트워크 모드
      // allowPrivateNetwork: true, // 레거시 별칭
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // 레거시 단일 프로파일 재정의
    remoteCdpTimeoutMs: 1500, // 원격 CDP HTTP 타임아웃 (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // 원격 CDP WebSocket 핸드셰이크 타임아웃 (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

참고:

- 브라우저 제어 서비스는 `gateway.port`에서 파생된 포트로 루프백에 바인딩됩니다 (기본값: `18791`, 게이트웨이 + 2). 릴레이는 다음 포트 (`18792`)를 사용합니다.
- 게이트웨이 포트를 재정의하면 (`gateway.port` 또는 `OPENCLAW_GATEWAY_PORT`), 파생된 브라우저 포트도 같은 "패밀리"에 머물도록 이동합니다.
- `cdpUrl`은 설정되지 않으면 릴레이 포트로 기본 설정됩니다.
- `remoteCdpTimeoutMs`는 원격 (비 루프백) CDP 접근성 검사에 적용됩니다.
- `remoteCdpHandshakeTimeoutMs`는 원격 CDP WebSocket 접근성 검사에 적용됩니다.
- 브라우저 탐색/탭 열기는 탐색 전 SSRF 가드되며, 탐색 후 최종 `http(s)` URL에서 최선 노력으로 재확인됩니다.
- `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`는 기본값이 `true`입니다 (신뢰 네트워크 모델). 엄격한 공개 전용 브라우징을 위해 `false`로 설정하세요.
- `browser.ssrfPolicy.allowPrivateNetwork`는 호환성을 위한 레거시 별칭으로 계속 지원됩니다.
- `attachOnly: true`는 "로컬 브라우저를 실행하지 않고, 이미 실행 중인 경우에만 연결"을 의미합니다.
- `color` + 프로파일별 `color`는 어떤 프로파일이 활성 상태인지 확인할 수 있도록 브라우저 UI를 색칠합니다.
- 기본 프로파일은 `chrome` (확장 프로그램 릴레이)입니다. 관리형 브라우저를 사용하려면 `defaultProfile: "openclaw"`를 사용하세요.
- 자동 감지 순서: Chromium 기반인 경우 시스템 기본 브라우저; 그렇지 않으면 Chrome → Brave → Edge → Chromium → Chrome Canary.
- 로컬 `openclaw` 프로파일은 `cdpPort`/`cdpUrl`을 자동 할당합니다 — 원격 CDP에만 설정하세요.

## Brave (또는 다른 Chromium 기반 브라우저) 사용

**시스템 기본** 브라우저가 Chromium 기반 (Chrome/Brave/Edge 등)이면,
OpenClaw가 자동으로 사용합니다. 자동 감지를 재정의하려면 `browser.executablePath`를
설정하세요:

CLI 예시:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## 로컬 vs 원격 제어

- **로컬 제어 (기본값):** 게이트웨이가 루프백 제어 서비스를 시작하고 로컬 브라우저를 실행할 수 있습니다.
- **원격 제어 (노드 호스트):** 브라우저가 있는 머신에서 노드 호스트를 실행합니다; 게이트웨이가 브라우저 액션을 프록시합니다.
- **원격 CDP:** `browser.profiles.<name>.cdpUrl` (또는 `browser.cdpUrl`)을 설정하여 원격 Chromium 기반 브라우저에 연결합니다. 이 경우 OpenClaw는 로컬 브라우저를 실행하지 않습니다.

원격 CDP URL에 인증을 포함할 수 있습니다:

- 쿼리 토큰 (예: `https://provider.example?token=<token>`)
- HTTP Basic 인증 (예: `https://user:pass@provider.example`)

OpenClaw는 `/json/*` 엔드포인트를 호출하고 CDP WebSocket에 연결할 때 인증을 유지합니다. 설정 파일에 토큰을 커밋하는 대신 환경 변수나 비밀 관리자를 사용하는 것이 좋습니다.

## 노드 브라우저 프록시 (제로 설정 기본값)

브라우저가 있는 머신에서 **노드 호스트**를 실행하면, OpenClaw가 추가 브라우저 설정 없이 해당 노드로 브라우저 도구 호출을 자동 라우팅할 수 있습니다. 이것이 원격 게이트웨이의 기본 경로입니다.

참고:

- 노드 호스트는 **프록시 명령**을 통해 로컬 브라우저 제어 서버를 노출합니다.
- 프로파일은 노드 자체의 `browser.profiles` 설정에서 가져옵니다 (로컬과 동일).
- 원하지 않으면 비활성화:
  - 노드에서: `nodeHost.browserProxy.enabled=false`
  - 게이트웨이에서: `gateway.nodes.browser.mode="off"`

## Browserless (호스팅 원격 CDP)

[Browserless](https://browserless.io)는 HTTPS를 통해 CDP 엔드포인트를 노출하는 호스팅 Chromium 서비스입니다. OpenClaw 브라우저 프로파일을 Browserless 리전 엔드포인트로 지정하고 API 키로 인증할 수 있습니다.

예시:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

참고:

- `<BROWSERLESS_API_KEY>`를 실제 Browserless 토큰으로 교체하세요.
- Browserless 계정에 맞는 리전 엔드포인트를 선택하세요 (해당 문서 참조).

## 보안

핵심 개념:

- 브라우저 제어는 루프백 전용입니다; 접근은 게이트웨이의 인증 또는 노드 페어링을 통해 이루어집니다.
- 브라우저 제어가 활성화되어 있고 인증이 설정되지 않은 경우, OpenClaw는 시작 시 `gateway.auth.token`을 자동 생성하고 설정에 저장합니다.
- 게이트웨이와 노드 호스트를 사설 네트워크 (Tailscale)에 유지하세요; 공개 노출을 피하세요.
- 원격 CDP URL/토큰을 비밀로 취급하세요; 환경 변수나 비밀 관리자를 사용하세요.

원격 CDP 팁:

- 가능하면 HTTPS 엔드포인트와 단기 토큰을 사용하세요.
- 설정 파일에 장기 토큰을 직접 포함하지 마세요.

## 프로파일 (다중 브라우저)

OpenClaw는 여러 명명된 프로파일 (라우팅 설정)을 지원합니다. 프로파일은 다음과 같을 수 있습니다:

- **openclaw 관리형**: 자체 사용자 데이터 디렉토리 + CDP 포트를 가진 전용 Chromium 기반 브라우저 인스턴스
- **원격**: 명시적 CDP URL (다른 곳에서 실행 중인 Chromium 기반 브라우저)
- **확장 프로그램 릴레이**: 로컬 릴레이 + Chrome 확장 프로그램을 통한 기존 Chrome 탭

기본값:

- `openclaw` 프로파일이 없으면 자동 생성됩니다.
- `chrome` 프로파일은 Chrome 확장 프로그램 릴레이용으로 내장되어 있습니다 (기본적으로 `http://127.0.0.1:18792`를 가리킴).
- 로컬 CDP 포트는 기본적으로 **18800-18899**에서 할당됩니다.
- 프로파일을 삭제하면 로컬 데이터 디렉토리가 휴지통으로 이동됩니다.

모든 제어 엔드포인트는 `?profile=<name>`을 받습니다; CLI는 `--browser-profile`을 사용합니다.

## Chrome 확장 프로그램 릴레이 (기존 Chrome 사용)

OpenClaw는 로컬 CDP 릴레이 + Chrome 확장 프로그램을 통해 별도의 "openclaw" Chrome 인스턴스 없이 **기존 Chrome 탭**을 구동할 수도 있습니다.

전체 가이드: [Chrome 확장 프로그램](/tools/chrome-extension)

흐름:

- 게이트웨이가 로컬에서 실행되거나 (같은 머신) 브라우저 머신에서 노드 호스트가 실행됩니다.
- 로컬 **릴레이 서버**가 루프백 `cdpUrl`에서 수신합니다 (기본값: `http://127.0.0.1:18792`).
- 탭에서 **OpenClaw Browser Relay** 확장 프로그램 아이콘을 클릭하여 연결합니다 (자동 연결 없음).
- 에이전트는 올바른 프로파일을 선택하여 일반 `browser` 도구를 통해 해당 탭을 제어합니다.

게이트웨이가 다른 곳에서 실행되면, 브라우저 머신에서 노드 호스트를 실행하여 게이트웨이가 브라우저 액션을 프록시할 수 있게 하세요.

### 샌드박스 세션

에이전트 세션이 샌드박스되어 있으면, `browser` 도구는 기본적으로 `target="sandbox"` (샌드박스 브라우저)를 사용할 수 있습니다.
Chrome 확장 프로그램 릴레이 인수인계는 호스트 브라우저 제어가 필요하므로:

- 세션을 샌드박스 없이 실행하거나,
- `agents.defaults.sandbox.browser.allowHostControl: true`를 설정하고 도구 호출 시 `target="host"`를 사용하세요.

### 설정

1. 확장 프로그램 로드 (개발/언패킹):

```bash
openclaw browser extension install
```

- Chrome → `chrome://extensions` → "개발자 모드" 활성화
- "압축해제된 확장 프로그램을 로드합니다" → `openclaw browser extension path`에서 출력된 디렉토리 선택
- 확장 프로그램을 고정한 후, 제어하려는 탭에서 클릭 (배지에 `ON` 표시).

2. 사용:

- CLI: `openclaw browser --browser-profile chrome tabs`
- 에이전트 도구: `browser` with `profile="chrome"`

선택 사항: 다른 이름이나 릴레이 포트를 원하면 자체 프로파일을 생성하세요:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

참고:

- 이 모드는 대부분의 작업 (스크린샷/스냅샷/액션)에 Playwright-on-CDP를 사용합니다.
- 확장 프로그램 아이콘을 다시 클릭하여 분리합니다.

## 격리 보장

- **전용 사용자 데이터 디렉토리**: 개인 브라우저 프로파일을 절대 건드리지 않습니다.
- **전용 포트**: 개발 워크플로우와의 충돌을 방지하기 위해 `9222`를 피합니다.
- **결정론적 탭 제어**: "마지막 탭"이 아닌 `targetId`로 탭을 대상으로 합니다.

## 브라우저 선택

로컬에서 실행할 때, OpenClaw는 사용 가능한 첫 번째 브라우저를 선택합니다:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

`browser.executablePath`로 재정의할 수 있습니다.

플랫폼:

- macOS: `/Applications`와 `~/Applications`을 확인합니다.
- Linux: `google-chrome`, `brave`, `microsoft-edge`, `chromium` 등을 찾습니다.
- Windows: 일반적인 설치 위치를 확인합니다.

## 제어 API (선택 사항)

로컬 통합 전용으로, 게이트웨이는 작은 루프백 HTTP API를 노출합니다:

- 상태/시작/중지: `GET /`, `POST /start`, `POST /stop`
- 탭: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
- 스냅샷/스크린샷: `GET /snapshot`, `POST /screenshot`
- 액션: `POST /navigate`, `POST /act`
- 후크: `POST /hooks/file-chooser`, `POST /hooks/dialog`
- 다운로드: `POST /download`, `POST /wait/download`
- 디버깅: `GET /console`, `POST /pdf`
- 디버깅: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
- 네트워크: `POST /response/body`
- 상태: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
- 상태: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
- 설정: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

모든 엔드포인트는 `?profile=<name>`을 받습니다.

게이트웨이 인증이 설정된 경우, 브라우저 HTTP 경로에도 인증이 필요합니다:

- `Authorization: Bearer <gateway token>`
- `x-openclaw-password: <gateway password>` 또는 해당 비밀번호로 HTTP Basic 인증

### Playwright 요구 사항

일부 기능 (navigate/act/AI snapshot/role snapshot, 요소 스크린샷, PDF)은 Playwright가 필요합니다. Playwright가 설치되지 않은 경우, 해당 엔드포인트는 명확한 501 오류를 반환합니다. ARIA 스냅샷과 기본 스크린샷은 openclaw 관리형 Chrome에서 여전히 작동합니다. Chrome 확장 프로그램 릴레이 드라이버의 경우, ARIA 스냅샷과 스크린샷에 Playwright가 필요합니다.

`Playwright is not available in this gateway build`가 표시되면, 전체 Playwright 패키지 (`playwright-core`가 아닌)를 설치하고 게이트웨이를 재시작하거나, 브라우저 지원과 함께 OpenClaw를 재설치하세요.

#### Docker Playwright 설치

게이트웨이가 Docker에서 실행되는 경우, `npx playwright` (npm 재정의 충돌)를 피하세요.
대신 번들된 CLI를 사용하세요:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

브라우저 다운로드를 유지하려면, `PLAYWRIGHT_BROWSERS_PATH` (예: `/home/node/.cache/ms-playwright`)를 설정하고 `/home/node`가 `OPENCLAW_HOME_VOLUME` 또는 바인드 마운트를 통해 유지되도록 하세요. [Docker](/install/docker)를 참조하세요.

## 작동 방식 (내부)

높은 수준의 흐름:

- 작은 **제어 서버**가 HTTP 요청을 받습니다.
- **CDP**를 통해 Chromium 기반 브라우저 (Chrome/Brave/Edge/Chromium)에 연결합니다.
- 고급 액션 (클릭/입력/스냅샷/PDF)의 경우, CDP 위에 **Playwright**를 사용합니다.
- Playwright가 없으면, 비 Playwright 작업만 사용 가능합니다.

이 설계는 에이전트를 안정적이고 결정론적인 인터페이스에 유지하면서 로컬/원격 브라우저와 프로파일을 교체할 수 있게 합니다.

## CLI 빠른 참조

모든 명령은 `--browser-profile <name>`을 받아 특정 프로파일을 대상으로 합니다.
모든 명령은 기계 판독 가능한 출력 (안정적인 페이로드)을 위해 `--json`도 받습니다.

기본:

- `openclaw browser status`
- `openclaw browser start`
- `openclaw browser stop`
- `openclaw browser tabs`
- `openclaw browser tab`
- `openclaw browser tab new`
- `openclaw browser tab select 2`
- `openclaw browser tab close 2`
- `openclaw browser open https://example.com`
- `openclaw browser focus abcd1234`
- `openclaw browser close abcd1234`

검사:

- `openclaw browser screenshot`
- `openclaw browser screenshot --full-page`
- `openclaw browser screenshot --ref 12`
- `openclaw browser screenshot --ref e12`
- `openclaw browser snapshot`
- `openclaw browser snapshot --format aria --limit 200`
- `openclaw browser snapshot --interactive --compact --depth 6`
- `openclaw browser snapshot --efficient`
- `openclaw browser snapshot --labels`
- `openclaw browser snapshot --selector "#main" --interactive`
- `openclaw browser snapshot --frame "iframe#main" --interactive`
- `openclaw browser console --level error`
- `openclaw browser errors --clear`
- `openclaw browser requests --filter api --clear`
- `openclaw browser pdf`
- `openclaw browser responsebody "**/api" --max-chars 5000`

액션:

- `openclaw browser navigate https://example.com`
- `openclaw browser resize 1280 720`
- `openclaw browser click 12 --double`
- `openclaw browser click e12 --double`
- `openclaw browser type 23 "hello" --submit`
- `openclaw browser press Enter`
- `openclaw browser hover 44`
- `openclaw browser scrollintoview e12`
- `openclaw browser drag 10 11`
- `openclaw browser select 9 OptionA OptionB`
- `openclaw browser download e12 report.pdf`
- `openclaw browser waitfordownload report.pdf`
- `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
- `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
- `openclaw browser dialog --accept`
- `openclaw browser wait --text "Done"`
- `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
- `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
- `openclaw browser highlight e12`
- `openclaw browser trace start`
- `openclaw browser trace stop`

상태:

- `openclaw browser cookies`
- `openclaw browser cookies set session abc123 --url "https://example.com"`
- `openclaw browser cookies clear`
- `openclaw browser storage local get`
- `openclaw browser storage local set theme dark`
- `openclaw browser storage session clear`
- `openclaw browser set offline on`
- `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
- `openclaw browser set credentials user pass`
- `openclaw browser set credentials --clear`
- `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
- `openclaw browser set geo --clear`
- `openclaw browser set media dark`
- `openclaw browser set timezone America/New_York`
- `openclaw browser set locale en-US`
- `openclaw browser set device "iPhone 14"`

참고:

- `upload`과 `dialog`는 **무장** 호출입니다; 선택기/대화상자를 트리거하는 클릭/프레스 전에 실행하세요.
- 다운로드 및 트레이스 출력 경로는 OpenClaw 임시 루트로 제한됩니다:
  - 트레이스: `/tmp/openclaw` (폴백: `${os.tmpdir()}/openclaw`)
  - 다운로드: `/tmp/openclaw/downloads` (폴백: `${os.tmpdir()}/openclaw/downloads`)
- 업로드 경로는 OpenClaw 임시 업로드 루트로 제한됩니다:
  - 업로드: `/tmp/openclaw/uploads` (폴백: `${os.tmpdir()}/openclaw/uploads`)
- `upload`은 `--input-ref` 또는 `--element`를 통해 파일 입력을 직접 설정할 수도 있습니다.
- `snapshot`:
  - `--format ai` (Playwright 설치 시 기본값): 숫자 참조가 있는 AI 스냅샷을 반환합니다 (`aria-ref="<n>"`).
  - `--format aria`: 접근성 트리를 반환합니다 (참조 없음; 검사 전용).
  - `--efficient` (또는 `--mode efficient`): 컴팩트 역할 스냅샷 프리셋 (interactive + compact + depth + 낮은 maxChars).
  - 설정 기본값 (도구/CLI 전용): 호출자가 모드를 전달하지 않을 때 효율적인 스냅샷을 사용하려면 `browser.snapshotDefaults.mode: "efficient"`를 설정하세요 ([게이트웨이 설정](/gateway/configuration#browser-openclaw-managed-browser) 참조).
  - 역할 스냅샷 옵션 (`--interactive`, `--compact`, `--depth`, `--selector`)은 `ref=e12`와 같은 참조가 있는 역할 기반 스냅샷을 강제합니다.
  - `--frame "<iframe selector>"`는 역할 스냅샷을 iframe으로 범위를 지정합니다 (`e12`와 같은 역할 참조와 함께).
  - `--interactive`는 인터랙티브 요소의 평면적이고 선택하기 쉬운 목록을 출력합니다 (액션 구동에 최적).
  - `--labels`는 참조 레이블이 오버레이된 뷰포트 전용 스크린샷을 추가합니다 (`MEDIA:<path>` 출력).
- `click`/`type` 등은 `snapshot`의 `ref`가 필요합니다 (숫자 `12` 또는 역할 참조 `e12`).
  액션에 CSS 선택자는 의도적으로 지원되지 않습니다.

## 스냅샷과 참조

OpenClaw는 두 가지 "스냅샷" 스타일을 지원합니다:

- **AI 스냅샷 (숫자 참조)**: `openclaw browser snapshot` (기본값; `--format ai`)
  - 출력: 숫자 참조가 포함된 텍스트 스냅샷.
  - 액션: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  - 내부적으로 참조는 Playwright의 `aria-ref`를 통해 해결됩니다.

- **역할 스냅샷 (`e12`와 같은 역할 참조)**: `openclaw browser snapshot --interactive` (또는 `--compact`, `--depth`, `--selector`, `--frame`)
  - 출력: `[ref=e12]` (그리고 선택적으로 `[nth=1]`)가 있는 역할 기반 목록/트리.
  - 액션: `openclaw browser click e12`, `openclaw browser highlight e12`.
  - 내부적으로 참조는 `getByRole(...)` (중복의 경우 `nth()` 추가)를 통해 해결됩니다.
  - `--labels`를 추가하면 `e12` 레이블이 오버레이된 뷰포트 스크린샷이 포함됩니다.

참조 동작:

- 참조는 **탐색 간에 안정적이지 않습니다**; 무언가 실패하면 `snapshot`을 다시 실행하고 새 참조를 사용하세요.
- 역할 스냅샷이 `--frame`으로 촬영된 경우, 다음 역할 스냅샷까지 역할 참조는 해당 iframe으로 범위가 지정됩니다.

## 대기 파워업

시간/텍스트 이상을 기다릴 수 있습니다:

- URL 대기 (Playwright에서 glob 지원):
  - `openclaw browser wait --url "**/dash"`
- 로드 상태 대기:
  - `openclaw browser wait --load networkidle`
- JS 술어 대기:
  - `openclaw browser wait --fn "window.ready===true"`
- 선택자가 보일 때까지 대기:
  - `openclaw browser wait "#main"`

이들을 결합할 수 있습니다:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## 디버그 워크플로우

액션이 실패할 때 (예: "not visible", "strict mode violation", "covered"):

1. `openclaw browser snapshot --interactive`
2. `click <ref>` / `type <ref>` 사용 (인터랙티브 모드에서 역할 참조 선호)
3. 여전히 실패하면: `openclaw browser highlight <ref>`로 Playwright가 대상으로 하는 것을 확인
4. 페이지가 이상하게 동작하면:
   - `openclaw browser errors --clear`
   - `openclaw browser requests --filter api --clear`
5. 심층 디버깅을 위해: 트레이스 녹화:
   - `openclaw browser trace start`
   - 문제를 재현
   - `openclaw browser trace stop` (`TRACE:<path>` 출력)

## JSON 출력

`--json`은 스크립팅과 구조화된 도구를 위한 것입니다.

예시:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

JSON의 역할 스냅샷에는 `refs`와 작은 `stats` 블록 (lines/chars/refs/interactive)이 포함되어 도구가 페이로드 크기와 밀도를 추론할 수 있습니다.

## 상태 및 환경 조작

"사이트가 X처럼 동작하게 만들기" 워크플로우에 유용합니다:

- 쿠키: `cookies`, `cookies set`, `cookies clear`
- 스토리지: `storage local|session get|set|clear`
- 오프라인: `set offline on|off`
- 헤더: `set headers --headers-json '{"X-Debug":"1"}'` (레거시 `set headers --json '{"X-Debug":"1"}'`도 계속 지원)
- HTTP basic 인증: `set credentials user pass` (또는 `--clear`)
- 지오로케이션: `set geo <lat> <lon> --origin "https://example.com"` (또는 `--clear`)
- 미디어: `set media dark|light|no-preference|none`
- 타임존 / 로케일: `set timezone ...`, `set locale ...`
- 디바이스 / 뷰포트:
  - `set device "iPhone 14"` (Playwright 디바이스 프리셋)
  - `set viewport 1280 720`

## 보안 및 개인 정보

- openclaw 브라우저 프로파일에는 로그인된 세션이 포함될 수 있습니다; 민감하게 취급하세요.
- `browser act kind=evaluate` / `openclaw browser evaluate` 및 `wait --fn`은 페이지 컨텍스트에서 임의의 JavaScript를 실행합니다. 프롬프트 인젝션이 이를 조종할 수 있습니다. 필요하지 않으면 `browser.evaluateEnabled=false`로 비활성화하세요.
- 로그인 및 안티봇 참고 (X/Twitter 등)는 [브라우저 로그인 + X/Twitter 게시](/tools/browser-login)를 참조하세요.
- 게이트웨이/노드 호스트를 비공개로 유지하세요 (루프백 또는 tailnet 전용).
- 원격 CDP 엔드포인트는 강력합니다; 터널링하고 보호하세요.

엄격 모드 예시 (기본적으로 사설/내부 대상 차단):

```json5
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // 선택적 정확한 허용
    },
  },
}
```

## 문제 해결

Linux 관련 문제 (특히 snap Chromium)는 [브라우저 문제 해결](/tools/browser-linux-troubleshooting)을 참조하세요.

## 에이전트 도구 + 제어 방식

에이전트는 브라우저 자동화를 위한 **하나의 도구**를 받습니다:

- `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

매핑 방식:

- `browser snapshot`은 안정적인 UI 트리 (AI 또는 ARIA)를 반환합니다.
- `browser act`은 스냅샷 `ref` ID를 사용하여 클릭/입력/드래그/선택합니다.
- `browser screenshot`은 픽셀을 캡처합니다 (전체 페이지 또는 요소).
- `browser`는 다음을 받습니다:
  - `profile`로 명명된 브라우저 프로파일을 선택 (openclaw, chrome 또는 원격 CDP).
  - `target` (`sandbox` | `host` | `node`)로 브라우저가 있는 위치를 선택.
  - 샌드박스 세션에서 `target: "host"`는 `agents.defaults.sandbox.browser.allowHostControl=true`가 필요합니다.
  - `target`이 생략된 경우: 샌드박스 세션은 `sandbox`가 기본값, 비 샌드박스 세션은 `host`가 기본값입니다.
  - 브라우저 지원 노드가 연결된 경우, `target="host"` 또는 `target="node"`를 고정하지 않으면 도구가 자동으로 라우팅할 수 있습니다.

이는 에이전트를 결정론적으로 유지하고 불안정한 선택자를 피합니다.
