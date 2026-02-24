---
summary: "계획: 모든 메시징 커넥터를 위한 하나의 깔끔한 플러그인 SDK + 런타임"
read_when:
  - Defining or refactoring the plugin architecture
  - Migrating channel connectors to the plugin SDK/runtime
title: "Plugin SDK 리팩토링"
---

# Plugin SDK + 런타임 리팩토링 계획

목표: 모든 메시징 커넥터가 하나의 안정적인 API를 사용하는 플러그인(번들 또는 외부)이 됨.
어떤 플러그인도 `src/**`에서 직접 import하지 않음. 모든 의존성은 SDK 또는 런타임을 통해 전달.

## 왜 지금인가

- 현재 커넥터는 패턴이 혼합됨: 직접 코어 import, dist 전용 브리지, 커스텀 헬퍼.
- 이것이 업그레이드를 취약하게 만들고 깔끔한 외부 플러그인 표면을 차단함.

## 대상 아키텍처 (두 레이어)

### 1) Plugin SDK (컴파일 타임, 안정적, 게시 가능)

범위: 타입, 헬퍼, 구성 유틸리티. 런타임 상태 없음, 부작용 없음.

내용 (예시):

- 타입: `ChannelPlugin`, 어댑터, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
- 구성 헬퍼: `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
- 페어링 헬퍼: `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
- 온보딩 헬퍼: `promptChannelAccessConfig`, `addWildcardAllowFrom`, 온보딩 타입.
- 도구 파라미터 헬퍼: `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
- 문서 링크 헬퍼: `formatDocsLink`.

전달:

- `openclaw/plugin-sdk`로 게시 (또는 코어에서 `openclaw/plugin-sdk` 하위로 export).
- 명시적 안정성 보장이 있는 semver.

### 2) Plugin Runtime (실행 표면, 주입됨)

범위: 코어 런타임 동작에 접촉하는 모든 것.
플러그인이 `src/**`를 import하지 않도록 `OpenClawPluginApi.runtime`을 통해 접근.

제안된 표면 (최소하지만 완전):

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: {
            text?: string;
            mediaUrls?: string[];
            mediaUrl?: string;
          }) => void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // Teams 스타일 흐름을 위한 어댑터
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: RoutePeerKind; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
      ): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```

참고:

- 런타임이 코어 동작에 접근하는 유일한 방법.
- SDK는 의도적으로 작고 안정적.
- 각 런타임 메서드는 기존 코어 구현에 매핑됨 (중복 없음).

## 마이그레이션 계획 (단계적, 안전)

### Phase 0: 스캐폴딩

- `openclaw/plugin-sdk` 도입.
- 위의 표면으로 `OpenClawPluginApi`에 `api.runtime` 추가.
- 전환 기간 동안 기존 import 유지 (사용 중단 경고).

### Phase 1: 브리지 정리 (저위험)

- 확장별 `core-bridge.ts`를 `api.runtime`으로 교체.
- BlueBubbles, Zalo, Zalo Personal 먼저 마이그레이션 (이미 가까움).
- 중복된 브리지 코드 제거.

### Phase 2: 가벼운 직접 import 플러그인

- Matrix를 SDK + 런타임으로 마이그레이션.
- 온보딩, 디렉토리, 그룹 멘션 로직 검증.

### Phase 3: 무거운 직접 import 플러그인

- MS Teams 마이그레이션 (가장 큰 런타임 헬퍼 세트).
- reply/typing 의미론이 현재 동작과 일치하는지 확인.

### Phase 4: iMessage 플러그인화

- iMessage를 `extensions/imessage`로 이동.
- 직접 코어 호출을 `api.runtime`으로 교체.
- 구성 키, CLI 동작, 문서를 그대로 유지.

### Phase 5: 시행

- lint 규칙 / CI 검사 추가: `extensions/**`에서 `src/**` import 없음.
- 플러그인 SDK/버전 호환성 검사 추가 (런타임 + SDK semver).

## 호환성 및 버전 관리

- SDK: semver, 게시, 문서화된 변경.
- 런타임: 코어 릴리스별 버전 관리. `api.runtime.version` 추가.
- 플러그인은 필요한 런타임 범위를 선언 (예: `openclawRuntime: ">=2026.2.0"`).

## 테스트 전략

- 어댑터 수준 단위 테스트 (실제 코어 구현으로 런타임 함수 실행).
- 플러그인별 골든 테스트: 동작 드리프트가 없는지 확인 (라우팅, 페어링, 허용 목록, 멘션 게이팅).
- CI에서 사용되는 단일 엔드투엔드 플러그인 샘플 (설치 + 실행 + 스모크).

## 미해결 질문

- SDK 타입 호스팅 위치: 별도 패키지 또는 코어 export?
- 런타임 타입 배포: SDK(타입만) 또는 코어?
- 번들 vs 외부 플러그인에 대한 문서 링크 노출 방법?
- 전환 기간 동안 저장소 내 플러그인에 대해 제한적인 직접 코어 import를 허용할 것인가?

## 성공 기준

- 모든 채널 커넥터가 SDK + 런타임을 사용하는 플러그인.
- `extensions/**`에서 `src/**` import 없음.
- 새 커넥터 템플릿이 SDK + 런타임에만 의존.
- 외부 플러그인을 코어 소스 접근 없이 개발 및 업데이트 가능.

관련 문서: [Plugins](/tools/plugin), [Channels](/channels/index), [Configuration](/gateway/configuration).
