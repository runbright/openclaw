---
summary: "OpenClaw 플러그인/확장 기능: 검색, 구성 및 안전성"
read_when:
  - 플러그인/확장 기능을 추가하거나 수정하는 경우
  - 플러그인 설치 또는 로드 규칙을 문서화하는 경우
title: "플러그인"
---

# 플러그인 (확장 기능)

## 빠른 시작 (플러그인이 처음이신가요?)

플러그인은 OpenClaw에 추가 기능(명령, 도구, Gateway RPC)을 확장하는 **작은 코드 모듈**입니다.

대부분의 경우 핵심 OpenClaw에 아직 내장되지 않은 기능이 필요하거나 선택적 기능을 메인 설치에서 분리하고 싶을 때 플러그인을 사용합니다.

빠른 경로:

1. 이미 로드된 항목 확인:

```bash
openclaw plugins list
```

2. 공식 플러그인 설치(예: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

npm 사양은 **레지스트리 전용**(패키지 이름 + 선택적 버전/태그)입니다. Git/URL/파일 사양은 거부됩니다.

3. Gateway를 재시작한 다음 `plugins.entries.<id>.config`에서 구성합니다.

구체적인 플러그인 예제는 [Voice Call](/plugins/voice-call)을 참조하세요.
서드파티 목록을 찾고 계신가요? [커뮤니티 플러그인](/plugins/community)을 참조하세요.

## 사용 가능한 플러그인 (공식)

- Microsoft Teams는 2026.1.15부터 플러그인 전용입니다; Teams를 사용하는 경우 `@openclaw/msteams`를 설치하세요.
- Memory (Core) -- 번들된 메모리 검색 플러그인(기본적으로 `plugins.slots.memory`를 통해 활성화)
- Memory (LanceDB) -- 번들된 장기 메모리 플러그인(자동 리콜/캡처; `plugins.slots.memory = "memory-lancedb"` 설정)
- [Voice Call](/plugins/voice-call) -- `@openclaw/voice-call`
- [Zalo Personal](/plugins/zalouser) -- `@openclaw/zalouser`
- [Matrix](/channels/matrix) -- `@openclaw/matrix`
- [Nostr](/channels/nostr) -- `@openclaw/nostr`
- [Zalo](/channels/zalo) -- `@openclaw/zalo`
- [Microsoft Teams](/channels/msteams) -- `@openclaw/msteams`
- Google Antigravity OAuth (프로바이더 인증) -- `google-antigravity-auth`로 번들됨(기본 비활성화)
- Gemini CLI OAuth (프로바이더 인증) -- `google-gemini-cli-auth`로 번들됨(기본 비활성화)
- Qwen OAuth (프로바이더 인증) -- `qwen-portal-auth`로 번들됨(기본 비활성화)
- Copilot Proxy (프로바이더 인증) -- 로컬 VS Code Copilot Proxy 브릿지; 내장 `github-copilot` 디바이스 로그인과 별개(번들됨, 기본 비활성화)

OpenClaw 플러그인은 jiti를 통해 런타임에 로드되는 **TypeScript 모듈**입니다. **구성 유효성 검사는 플러그인 코드를 실행하지 않으며** 플러그인 매니페스트와 JSON Schema를 대신 사용합니다. [플러그인 매니페스트](/plugins/manifest)를 참조하세요.

플러그인이 등록할 수 있는 항목:

- Gateway RPC 메서드
- Gateway HTTP 핸들러
- 에이전트 도구
- CLI 명령
- 백그라운드 서비스
- 선택적 구성 유효성 검사
- **스킬**(플러그인 매니페스트에 `skills` 디렉토리를 나열하여)
- **자동 응답 명령**(AI 에이전트를 호출하지 않고 실행)

플러그인은 Gateway와 **인프로세스**로 실행되므로 신뢰할 수 있는 코드로 취급하세요.
도구 작성 가이드: [플러그인 에이전트 도구](/plugins/agent-tools).

## 런타임 헬퍼

플러그인은 `api.runtime`을 통해 선택된 핵심 헬퍼에 접근할 수 있습니다. 전화 TTS의 경우:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

참고:

- 핵심 `messages.tts` 구성(OpenAI 또는 ElevenLabs)을 사용합니다.
- PCM 오디오 버퍼 + 샘플 레이트를 반환합니다. 플러그인은 프로바이더에 맞게 리샘플링/인코딩해야 합니다.
- Edge TTS는 전화 통화에 지원되지 않습니다.

## 검색 및 우선순위

OpenClaw는 다음 순서로 스캔합니다:

1. 구성 경로

- `plugins.load.paths`(파일 또는 디렉토리)

2. 워크스페이스 확장 기능

- `<workspace>/.openclaw/extensions/*.ts`
- `<workspace>/.openclaw/extensions/*/index.ts`

3. 전역 확장 기능

- `~/.openclaw/extensions/*.ts`
- `~/.openclaw/extensions/*/index.ts`

4. 번들된 확장 기능(OpenClaw과 함께 제공, **기본 비활성화**)

- `<openclaw>/extensions/*`

번들된 플러그인은 `plugins.entries.<id>.enabled` 또는 `openclaw plugins enable <id>`를 통해 명시적으로 활성화해야 합니다. 설치된 플러그인은 기본적으로 활성화되지만 같은 방법으로 비활성화할 수 있습니다.

보안 강화 참고:

- `plugins.allow`가 비어 있고 비번들 플러그인이 검색 가능한 경우, OpenClaw은 플러그인 id와 소스를 포함한 시작 경고를 기록합니다.
- 후보 경로는 검색 수락 전에 안전 검사를 거칩니다. OpenClaw는 다음 경우에 후보를 차단합니다:
  - 확장 기능 항목이 플러그인 루트 외부로 해석되는 경우(심볼릭 링크/경로 탐색 이스케이프 포함),
  - 플러그인 루트/소스 경로가 전체 쓰기 가능인 경우,
  - 비번들 플러그인의 경로 소유권이 의심스러운 경우(POSIX 소유자가 현재 uid 또는 root가 아닌 경우).
- 설치/로드 경로 출처가 없는 로드된 비번들 플러그인은 경고를 발생시켜 신뢰를 고정(`plugins.allow`)하거나 설치 추적(`plugins.installs`)을 할 수 있습니다.

각 플러그인은 루트에 `openclaw.plugin.json` 파일을 포함해야 합니다. 경로가 파일을 가리키는 경우 플러그인 루트는 파일의 디렉토리이며 매니페스트를 포함해야 합니다.

여러 플러그인이 동일한 id로 해석되면 위 순서에서 첫 번째 일치가 우선하고 낮은 우선순위 복사본은 무시됩니다.

### 패키지 팩

플러그인 디렉토리에 `openclaw.extensions`가 포함된 `package.json`이 있을 수 있습니다:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

각 항목은 플러그인이 됩니다. 팩에 여러 확장 기능이 나열되면 플러그인 id는 `name/<fileBase>`가 됩니다.

플러그인이 npm 의존성을 가져오는 경우 해당 디렉토리에서 `node_modules`가 사용 가능하도록 설치하세요(`npm install` / `pnpm install`).

보안 가드레일: 모든 `openclaw.extensions` 항목은 심볼릭 링크 해석 후 플러그인 디렉토리 내에 있어야 합니다. 패키지 디렉토리를 벗어나는 항목은 거부됩니다.

보안 참고: `openclaw plugins install`은 `npm install --ignore-scripts`(라이프사이클 스크립트 없음)로 플러그인 의존성을 설치합니다. 플러그인 의존성 트리를 "순수 JS/TS"로 유지하고 `postinstall` 빌드가 필요한 패키지를 피하세요.

### 채널 카탈로그 메타데이터

채널 플러그인은 `openclaw.channel`을 통해 온보딩 메타데이터를, `openclaw.install`을 통해 설치 힌트를 광고할 수 있습니다. 이를 통해 핵심 카탈로그를 데이터 없이 유지합니다.

예제:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Self-hosted chat via Nextcloud Talk webhook bots.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw은 **외부 채널 카탈로그**(예: MPM 레지스트리 내보내기)도 병합할 수 있습니다. 다음 경로 중 하나에 JSON 파일을 놓으세요:

- `~/.openclaw/mpm/plugins.json`
- `~/.openclaw/mpm/catalog.json`
- `~/.openclaw/plugins/catalog.json`

또는 `OPENCLAW_PLUGIN_CATALOG_PATHS`(또는 `OPENCLAW_MPM_CATALOG_PATHS`)를 하나 이상의 JSON 파일에 지정하세요(쉼표/세미콜론/`PATH` 구분자). 각 파일은 `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`를 포함해야 합니다.

## 플러그인 ID

기본 플러그인 id:

- 패키지 팩: `package.json`의 `name`
- 독립 파일: 파일 기본 이름(`~/.../voice-call.ts` → `voice-call`)

플러그인이 `id`를 내보내면 OpenClaw은 이를 사용하지만 구성된 id와 일치하지 않으면 경고합니다.

## 구성

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

필드:

- `enabled`: 마스터 토글(기본값: true)
- `allow`: 허용 목록(선택 사항)
- `deny`: 거부 목록(선택 사항; 거부가 우선)
- `load.paths`: 추가 플러그인 파일/디렉토리
- `entries.<id>`: 플러그인별 토글 + 구성

구성 변경은 **게이트웨이 재시작이 필요합니다**.

유효성 검사 규칙(엄격):

- `entries`, `allow`, `deny` 또는 `slots`의 알 수 없는 플러그인 id는 **오류**입니다.
- 플러그인 매니페스트가 채널 id를 선언하지 않는 한 알 수 없는 `channels.<id>` 키는 **오류**입니다.
- 플러그인 구성은 `openclaw.plugin.json`(`configSchema`)에 포함된 JSON Schema를 사용하여 검증됩니다.
- 플러그인이 비활성화된 경우 구성은 보존되고 **경고**가 발생합니다.

## 플러그인 슬롯(독점 카테고리)

일부 플러그인 카테고리는 **독점적**(한 번에 하나만 활성)입니다. `plugins.slots`를 사용하여 어떤 플러그인이 슬롯을 소유할지 선택하세요:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // 또는 메모리 플러그인을 비활성화하려면 "none"
    },
  },
}
```

여러 플러그인이 `kind: "memory"`를 선언하면 선택된 것만 로드됩니다. 나머지는 진단과 함께 비활성화됩니다.

## Control UI(스키마 + 라벨)

Control UI는 `config.schema`(JSON Schema + `uiHints`)를 사용하여 더 나은 폼을 렌더링합니다.

OpenClaw은 검색된 플러그인을 기반으로 런타임에 `uiHints`를 보강합니다:

- `plugins.entries.<id>` / `.enabled` / `.config`에 대한 플러그인별 라벨 추가
- 선택적 플러그인 제공 구성 필드 힌트를 다음 아래에 병합:
  `plugins.entries.<id>.config.<field>`

플러그인 구성 필드에 좋은 라벨/플레이스홀더를 표시하고(비밀을 민감한 것으로 표시하려면) 플러그인 매니페스트에서 JSON Schema와 함께 `uiHints`를 제공하세요.

예제:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Region", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # 로컬 파일/디렉토리를 ~/.openclaw/extensions/<id>에 복사
openclaw plugins install ./extensions/voice-call # 상대 경로 가능
openclaw plugins install ./plugin.tgz           # 로컬 tarball에서 설치
openclaw plugins install ./plugin.zip           # 로컬 zip에서 설치
openclaw plugins install -l ./extensions/voice-call # 개발용 링크(복사 없음)
openclaw plugins install @openclaw/voice-call # npm에서 설치
openclaw plugins install @openclaw/voice-call --pin # 정확한 해석된 name@version 저장
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update`는 `plugins.installs` 아래에 추적된 npm 설치에만 작동합니다.
저장된 무결성 메타데이터가 업데이트 간에 변경되면 OpenClaw은 경고하고 확인을 요청합니다(전역 `--yes`를 사용하여 프롬프트를 건너뛸 수 있습니다).

플러그인은 자체 최상위 명령을 등록할 수도 있습니다(예: `openclaw voicecall`).

## 플러그인 API (개요)

플러그인은 다음 중 하나를 내보냅니다:

- 함수: `(api) => { ... }`
- 객체: `{ id, name, configSchema, register(api) { ... } }`

## 플러그인 훅

플러그인은 런타임에 훅을 등록할 수 있습니다. 이를 통해 플러그인이 별도의 훅 팩 설치 없이 이벤트 기반 자동화를 번들할 수 있습니다.

### 예제

```ts
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // 훅 로직이 여기에 들어갑니다.
    },
    {
      name: "my-plugin.command-new",
      description: "Runs when /new is invoked",
    },
  );
}
```

참고:

- `api.registerHook(...)`을 통해 훅을 명시적으로 등록합니다.
- 훅 자격 규칙은 여전히 적용됩니다(OS/바이너리/환경/구성 요구 사항).
- 플러그인 관리 훅은 `openclaw hooks list`에서 `plugin:<id>`와 함께 표시됩니다.
- `openclaw hooks`를 통해 플러그인 관리 훅을 활성화/비활성화할 수 없습니다; 대신 플러그인을 활성화/비활성화하세요.

## 프로바이더 플러그인(모델 인증)

플러그인은 **모델 프로바이더 인증** 흐름을 등록할 수 있어 사용자가 OpenClaw 내에서 OAuth 또는 API 키 설정을 실행할 수 있습니다(외부 스크립트 불필요).

`api.registerProvider(...)`를 통해 프로바이더를 등록합니다. 각 프로바이더는 하나 이상의 인증 방법(OAuth, API 키, 디바이스 코드 등)을 노출합니다. 이러한 방법은 다음을 지원합니다:

- `openclaw models auth login --provider <id> [--method <id>]`

예제:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // OAuth 흐름을 실행하고 인증 프로필을 반환합니다.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

참고:

- `run`은 `prompter`, `runtime`, `openUrl` 및 `oauth.createVpsAwareHandlers` 헬퍼가 포함된 `ProviderAuthContext`를 받습니다.
- 기본 모델이나 프로바이더 구성을 추가해야 할 때 `configPatch`를 반환합니다.
- `--set-default`가 에이전트 기본값을 업데이트할 수 있도록 `defaultModel`을 반환합니다.

### 메시징 채널 등록

플러그인은 내장 채널(WhatsApp, Telegram 등)처럼 동작하는 **채널 플러그인**을 등록할 수 있습니다. 채널 구성은 `channels.<id>` 아래에 위치하며 채널 플러그인 코드에 의해 유효성이 검사됩니다.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

참고:

- 구성을 `channels.<id>`에 넣으세요(`plugins.entries`가 아님).
- `meta.label`은 CLI/UI 목록의 라벨에 사용됩니다.
- `meta.aliases`는 정규화 및 CLI 입력을 위한 대체 id를 추가합니다.
- `meta.preferOver`는 둘 다 구성된 경우 자동 활성화를 건너뛸 채널 id 목록입니다.
- `meta.detailLabel`과 `meta.systemImage`는 UI가 더 풍부한 채널 라벨/아이콘을 표시할 수 있게 합니다.

### 새 메시징 채널 작성(단계별)

모델 프로바이더가 아닌 **새 채팅 표면**("메시징 채널")이 필요할 때 이를 사용하세요.
모델 프로바이더 문서는 `/providers/*` 아래에 있습니다.

1. id + 구성 형태 선택

- 모든 채널 구성은 `channels.<id>` 아래에 위치합니다.
- 멀티 계정 설정에는 `channels.<id>.accounts.<accountId>`를 선호합니다.

2. 채널 메타데이터 정의

- `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb`가 CLI/UI 목록을 제어합니다.
- `meta.docsPath`는 `/channels/<id>` 같은 문서 페이지를 가리켜야 합니다.
- `meta.preferOver`는 플러그인이 다른 채널을 대체할 수 있게 합니다(자동 활성화가 이를 선호).
- `meta.detailLabel`과 `meta.systemImage`는 UI에서 세부 텍스트/아이콘에 사용됩니다.

3. 필수 어댑터 구현

- `config.listAccountIds` + `config.resolveAccount`
- `capabilities`(채팅 유형, 미디어, 스레드 등)
- `outbound.deliveryMode` + `outbound.sendText`(기본 전송용)

4. 필요에 따라 선택적 어댑터 추가

- `setup`(마법사), `security`(DM 정책), `status`(상태/진단)
- `gateway`(시작/중지/로그인), `mentions`, `threading`, `streaming`
- `actions`(메시지 액션), `commands`(네이티브 명령 동작)

5. 플러그인에서 채널 등록

- `api.registerChannel({ plugin })`

최소 구성 예제:

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

최소 채널 플러그인(아웃바운드 전용):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // 여기서 `text`를 채널에 전달합니다
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

플러그인을 로드하고(확장 기능 디렉토리 또는 `plugins.load.paths`), 게이트웨이를 재시작한 다음 구성에서 `channels.<id>`를 설정하세요.

### 에이전트 도구

전용 가이드를 참조하세요: [플러그인 에이전트 도구](/plugins/agent-tools).

### Gateway RPC 메서드 등록

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### CLI 명령 등록

```ts
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### 자동 응답 명령 등록

플러그인은 **AI 에이전트를 호출하지 않고** 실행되는 사용자 정의 슬래시 명령을 등록할 수 있습니다. 이는 토글 명령, 상태 확인 또는 LLM 처리가 필요 없는 빠른 작업에 유용합니다.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

명령 핸들러 컨텍스트:

- `senderId`: 발신자의 ID(사용 가능한 경우)
- `channel`: 명령이 전송된 채널
- `isAuthorizedSender`: 발신자가 인증된 사용자인지 여부
- `args`: 명령 뒤에 전달된 인수(`acceptsArgs: true`인 경우)
- `commandBody`: 전체 명령 텍스트
- `config`: 현재 OpenClaw 구성

명령 옵션:

- `name`: 명령 이름(앞의 `/` 없이)
- `description`: 명령 목록에 표시되는 도움말 텍스트
- `acceptsArgs`: 명령이 인수를 받는지 여부(기본값: false). false이고 인수가 제공되면 명령이 일치하지 않고 메시지가 다른 핸들러로 넘어갑니다
- `requireAuth`: 인증된 발신자가 필요한지 여부(기본값: true)
- `handler`: `{ text: string }`을 반환하는 함수(비동기 가능)

인증 및 인수가 있는 예제:

```ts
api.registerCommand({
  name: "setmode",
  description: "Set plugin mode",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

참고:

- 플러그인 명령은 내장 명령 및 AI 에이전트 **이전에** 처리됩니다
- 명령은 전역적으로 등록되며 모든 채널에서 작동합니다
- 명령 이름은 대소문자를 구분하지 않습니다(`/MyStatus`는 `/mystatus`와 일치)
- 명령 이름은 문자로 시작하고 문자, 숫자, 하이픈, 밑줄만 포함해야 합니다
- 예약된 명령 이름(`help`, `status`, `reset` 등)은 플러그인으로 재정의할 수 없습니다
- 플러그인 간 중복 명령 등록은 진단 오류와 함께 실패합니다

### 백그라운드 서비스 등록

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

## 명명 규칙

- Gateway 메서드: `pluginId.action`(예: `voicecall.status`)
- 도구: `snake_case`(예: `voice_call`)
- CLI 명령: kebab 또는 camel, 핵심 명령과 충돌 방지

## 스킬

플러그인은 레포에 스킬을 포함할 수 있습니다(`skills/<name>/SKILL.md`).
`plugins.entries.<id>.enabled`(또는 기타 구성 게이트)로 활성화하고 워크스페이스/관리 스킬 위치에 존재하는지 확인하세요.

## 배포 (npm)

권장 패키징:

- 메인 패키지: `openclaw`(이 레포)
- 플러그인: `@openclaw/*` 아래의 별도 npm 패키지(예: `@openclaw/voice-call`)

배포 계약:

- 플러그인 `package.json`은 하나 이상의 항목 파일이 포함된 `openclaw.extensions`를 포함해야 합니다.
- 항목 파일은 `.js` 또는 `.ts`일 수 있습니다(jiti가 런타임에 TS를 로드합니다).
- `openclaw plugins install <npm-spec>`은 `npm pack`을 사용하고 `~/.openclaw/extensions/<id>/`에 추출하며 구성에서 활성화합니다.
- 구성 키 안정성: 스코프 패키지는 `plugins.entries.*`에 대해 **비스코프** id로 정규화됩니다.

## 예제 플러그인: Voice Call

이 레포에는 음성 통화 플러그인(Twilio 또는 로그 폴백)이 포함되어 있습니다:

- 소스: `extensions/voice-call`
- 스킬: `skills/voice-call`
- CLI: `openclaw voicecall start|status`
- 도구: `voice_call`
- RPC: `voicecall.start`, `voicecall.status`
- 구성 (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from`(선택적 `statusCallbackUrl`, `twimlUrl`)
- 구성 (dev): `provider: "log"`(네트워크 없음)

설정 및 사용법은 [Voice Call](/plugins/voice-call) 및 `extensions/voice-call/README.md`를 참조하세요.

## 안전 참고 사항

플러그인은 Gateway와 인프로세스로 실행됩니다. 신뢰할 수 있는 코드로 취급하세요:

- 신뢰할 수 있는 플러그인만 설치하세요.
- `plugins.allow` 허용 목록을 선호하세요.
- 변경 후 Gateway를 재시작하세요.

## 플러그인 테스트

플러그인은 테스트를 포함할 수 있으며(포함해야 합니다):

- 레포 내 플러그인은 `src/**` 아래에 Vitest 테스트를 유지할 수 있습니다(예: `src/plugins/voice-call.plugin.test.ts`).
- 별도로 배포된 플러그인은 자체 CI(린트/빌드/테스트)를 실행하고 `openclaw.extensions`가 빌드된 진입점(`dist/index.js`)을 가리키는지 검증해야 합니다.
