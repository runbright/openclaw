# OpenClaw Agent 동작 원리 및 쿼리 처리 플로우

## 목차

1. [개요](#개요)
2. [전체 시퀀스 다이어그램](#전체-시퀀스-다이어그램)
3. [단계별 상세 설명](#단계별-상세-설명)
4. [Agentic Loop 상세](#agentic-loop-상세)
5. [시스템 프롬프트 구성](#시스템-프롬프트-구성)
6. [도구(Tool) 실행 파이프라인](#도구tool-실행-파이프라인)
   - [LLM-도구 연결 아키텍처](#llm-도구-연결-아키텍처)
7. [세션 및 컨텍스트 관리](#세션-및-컨텍스트-관리)
8. [에러 처리 및 Failover](#에러-처리-및-failover)
9. [실전 예시: Discord에서 "두바이 비행기표 최저가 알아봐줘"](#실전-예시-discord에서-두바이-비행기표-최저가-알아봐줘)
10. [쉬운 설명: 복합 여행 계획 요청의 동작 원리](#쉬운-설명-복합-여행-계획-요청의-동작-원리)
11. [핵심 파일 참조](#핵심-파일-참조)

---

## 개요

OpenClaw Agent는 사용자의 쿼리(메시지)를 받아 LLM(대형 언어 모델)과 도구 실행을 반복하며 작업을 수행하는 **Agentic AI 시스템**이다.

핵심 동작 원리:

1. **사용자 입력** → CLI, 웹, 메시징 채널(Telegram, Discord, Slack 등)을 통해 수신
2. **게이트웨이 라우팅** → 메시지를 적절한 에이전트 세션으로 라우팅
3. **시스템 프롬프트 구성** → 에이전트 정체성, 도구, 스킬, 컨텍스트 조합
4. **LLM 호출** → 스트리밍으로 응답 생성
5. **도구 실행 루프** → LLM이 도구 사용을 요청하면 실행 후 결과를 다시 LLM에 전달
6. **응답 전달** → 최종 응답을 원래 채널로 전송

---

## 전체 시퀀스 다이어그램

### 1. End-to-End 쿼리 처리 흐름

```mermaid
sequenceDiagram
    actor User as 사용자
    participant Channel as 채널<br/>(CLI/Web/Telegram/Discord)
    participant Gateway as 게이트웨이<br/>(HTTP/WebSocket)
    participant AgentHandler as Agent Handler<br/>(server-methods/agent.ts)
    participant AgentCmd as agentCommand()<br/>(commands/agent.ts)
    participant Runner as Embedded Pi Runner<br/>(pi-embedded-runner/run.ts)
    participant Attempt as runEmbeddedAttempt()<br/>(run/attempt.ts)
    participant SDK as Pi AI SDK<br/>(@mariozechner/pi-ai)
    participant LLM as LLM Provider<br/>(Anthropic/OpenAI/Google)
    participant Tools as 도구 실행기<br/>(pi-tools.ts)
    participant Session as 세션 저장소<br/>(~/.openclaw/sessions/)
    participant Delivery as 응답 전달기<br/>(delivery.ts)

    User->>Channel: 1. 메시지 전송
    Channel->>Gateway: 2. HTTP/WS 요청 (RPC: "agent")
    Gateway->>AgentHandler: 3. RPC 라우팅

    Note over AgentHandler: 검증, 중복제거, 세션 해석

    AgentHandler-->>Channel: 4. 즉시 응답 {runId, status: "accepted"}
    AgentHandler->>AgentCmd: 5. 비동기 실행 (fire-and-forget)

    AgentCmd->>Runner: 6. runEmbeddedPiAgent() 호출
    Runner->>Session: 7. 세션 히스토리 로드

    loop 외부 루프: Auth/Error Retry (최대 10회)
        Runner->>Attempt: 8. runEmbeddedAttempt() 호출

        Note over Attempt: 시스템 프롬프트 구성<br/>도구 등록<br/>이미지 로드<br/>히스토리 정제

        Attempt->>SDK: 9. createAgentSession() + streamSimple()

        loop 내부 루프: Tool Use Loop (LLM이 중단할 때까지)
            SDK->>LLM: 10. API 호출 (messages + tools + system prompt)
            LLM-->>SDK: 11. 스트리밍 응답 (text_delta + tool_use)

            alt tool_use 블록 존재
                SDK->>Tools: 12. 도구 실행 (read/write/exec 등)
                Tools-->>SDK: 13. 도구 실행 결과
                Note over SDK: tool_result를 메시지에 추가<br/>→ LLM 재호출
            else stop_reason = "end_turn"
                Note over SDK: 루프 종료
            end
        end

        SDK-->>Attempt: 14. 최종 응답 + 메타데이터
        Attempt->>Session: 15. 세션 트랜스크립트 저장
        Attempt-->>Runner: 16. 결과 반환

        alt 성공
            Note over Runner: 루프 탈출
        else 컨텍스트 오버플로우
            Note over Runner: 컴팩션 후 재시도
        else 인증/과금 오류
            Note over Runner: 다음 프로필로 전환 후 재시도
        end
    end

    Runner-->>AgentCmd: 17. EmbeddedPiRunResult 반환
    AgentCmd->>Delivery: 18. deliverAgentCommandResult()
    Delivery->>Channel: 19. 채널 API로 응답 전송
    Channel-->>User: 20. 최종 응답 수신
```

### 2. 게이트웨이 내부 처리 흐름

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant HTTP as HTTP Server<br/>(server-http.ts)
    participant RPC as RPC Router<br/>(server-methods.ts)
    participant Agent as Agent Handler<br/>(server-methods/agent.ts)
    participant IdempCache as 멱등성 캐시
    participant SessionStore as 세션 스토어
    participant AgentCmd as agentCommand()

    Client->>HTTP: POST /rpc { method: "agent", params: {...} }
    HTTP->>RPC: 메서드 라우팅
    RPC->>Agent: agentHandlers.agent(params)

    Agent->>Agent: 1. 파라미터 스키마 검증
    Agent->>IdempCache: 2. idempotencyKey 중복 체크
    alt 중복 요청
        IdempCache-->>Agent: 캐시된 결과 반환
        Agent-->>Client: 이전 결과 즉시 반환
    end

    Agent->>Agent: 3. 첨부파일 파싱 (이미지 추출)
    Agent->>Agent: 4. 채널/에이전트 ID 검증
    Agent->>SessionStore: 5. 세션 해석 및 로드/생성

    Note over Agent: /reset → 세션 초기화<br/>/new → 새 세션 생성<br/>일반 → 기존 세션 이어가기

    Agent->>SessionStore: 6. 세션 엔트리 업데이트
    Agent->>Agent: 7. 전달 계획 수립 (채널, 대상)
    Agent-->>Client: 8. 즉시 응답 {runId, status: "accepted"}
    Agent->>AgentCmd: 9. 비동기 agentCommand() 실행

    Note over AgentCmd: 백그라운드에서 에이전트 실행<br/>완료 시 콜백으로 최종 응답 전달
```

### 3. Agentic Tool-Use Loop 상세

```mermaid
sequenceDiagram
    participant Attempt as runEmbeddedAttempt()
    participant Session as Agent Session
    participant Stream as streamSimple()
    participant LLM as LLM API
    participant Handler as Event Handlers
    participant Tool as Tool Executor
    participant Chunker as Block Chunker

    Attempt->>Session: createAgentSession(tools, history, systemPrompt)
    Attempt->>Handler: subscribeEmbeddedPiSession() 이벤트 구독

    Attempt->>Stream: streamSimple(model, messages)

    Stream->>LLM: API 호출 (POST /v1/messages)
    LLM-->>Stream: event: message_start
    Stream->>Handler: emit(message_start)
    Handler->>Handler: handleMessageStart() - 상태 초기화

    loop 텍스트 스트리밍
        LLM-->>Stream: event: content_block_delta (text)
        Stream->>Handler: emit(message_update)
        Handler->>Chunker: 텍스트 축적 + <think> 태그 제거
        Chunker-->>Handler: onPartialReply(cleanText)
    end

    alt LLM이 도구 호출 요청
        LLM-->>Stream: event: content_block (tool_use)
        Stream->>Handler: emit(tool_execution_start)
        Handler->>Handler: handleToolExecutionStart(toolName, params)

        Stream->>Tool: 도구 실행 (예: exec "ls -la")
        Tool-->>Stream: 실행 결과 반환

        Stream->>Handler: emit(tool_execution_end)
        Handler->>Handler: handleToolExecutionEnd(result)
        Handler->>Handler: emitToolResult() / emitToolSummary()

        Note over Stream: tool_result를 messages에 추가
        Stream->>LLM: 다시 API 호출 (이전 대화 + tool_result)

        Note over LLM: LLM이 도구 결과를 보고<br/>추가 텍스트 생성 또는<br/>다른 도구 호출
    end

    LLM-->>Stream: event: message_stop (stop_reason: "end_turn")
    Stream->>Handler: emit(message_end)
    Handler->>Handler: handleMessageEnd() - 최종 텍스트 조합
    Handler->>Chunker: onBlockReplyFlush() - 마지막 블록 전송

    Stream->>Handler: emit(agent_end)
    Handler-->>Attempt: assistantTexts[] + toolMetas[] + usage 반환
```

### 4. 채널별 메시지 수신 흐름

```mermaid
sequenceDiagram
    actor User as 사용자
    participant TG as Telegram Bot API
    participant BotHandler as bot-handlers.ts
    participant MsgCtx as bot-message-context.ts
    participant Dispatch as bot-message-dispatch.ts
    participant AutoReply as auto-reply/
    participant AgentCmd as agentCommand()
    participant Delivery as deliverOutboundPayloads()

    User->>TG: 텔레그램 메시지 전송
    TG->>BotHandler: Webhook 수신 (Update 객체)

    BotHandler->>MsgCtx: 메시지 컨텍스트 구축
    Note over MsgCtx: 발신자 정보<br/>채팅/스레드 ID<br/>첨부파일<br/>답장 컨텍스트

    MsgCtx-->>Dispatch: MessageContext 반환

    Dispatch->>Dispatch: 세션 키 해석 (sender/chat 기반)
    Dispatch->>Dispatch: 라우팅 바인딩에서 에이전트 해석
    Dispatch->>AutoReply: dispatchReplyWithBufferedBlockDispatcher()

    AutoReply->>AgentCmd: agentCommand() 호출
    Note over AgentCmd: (위의 전체 에이전트 실행 흐름과 동일)
    AgentCmd-->>AutoReply: 응답 페이로드 반환

    AutoReply->>Delivery: 채널별 포맷팅 및 전송
    Delivery->>TG: bot.api.sendMessage()
    TG-->>User: 응답 메시지 수신
```

---

## 단계별 상세 설명

### Phase 1: 사용자 입력 수신

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **1-1** | 사용자가 CLI, 웹, 또는 메시징 채널을 통해 메시지 전송 | `src/entry.ts` |
| **1-2** | CLI의 경우 Commander로 인자 파싱 후 `agentCliCommand()` 호출 | `src/cli/program/register.agent.ts` |
| **1-3** | 채널의 경우 Webhook/WebSocket으로 게이트웨이에 도달 | `src/telegram/bot-handlers.ts` 등 |

**CLI 진입 경로:**
```
openclaw agent --message "코드 리뷰해줘" --local
  → entry.ts → run-main.ts → buildProgram() → register.agent.ts
  → agentCliCommand(opts)
    → --local이면 agentCommand() 직접 호출
    → 아니면 callGateway()로 게이트웨이 경유
```

**채널 진입 경로:**
```
텔레그램 메시지 수신
  → bot-handlers.ts (Webhook)
  → bot-message-context.ts (컨텍스트 구축)
  → bot-message-dispatch.ts (세션/에이전트 해석)
  → auto-reply/ (에이전트 실행 트리거)
```

---

### Phase 2: 게이트웨이 라우팅

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **2-1** | HTTP/WebSocket 요청 수신 및 인증 확인 | `src/gateway/server-http.ts` |
| **2-2** | RPC 메서드(`"agent"`)에 따라 핸들러 라우팅 | `src/gateway/server-methods.ts` |
| **2-3** | `idempotencyKey`로 중복 요청 필터링 | `server-methods/agent.ts` |
| **2-4** | 세션 키 해석 → 기존 세션 로드 또는 새로 생성 | `server-methods/agent.ts` |
| **2-5** | 즉시 `{runId, status: "accepted"}` 응답 → 비동기 실행 | `server-methods/agent.ts` |

**RPC 요청 파라미터:**
```typescript
{
  message: string           // 사용자 입력 텍스트
  idempotencyKey: string    // 중복 방지 키
  sessionKey?: string       // 세션 식별 (예: "agent@telegram:12345")
  agentId?: string          // 특정 에이전트 지정
  thinking?: string         // 추론 레벨 ("off"|"low"|"medium"|"high")
  deliver?: boolean         // 채널로 응답 전달 여부
  channel?: string          // 전달 채널 (telegram|discord|slack 등)
  attachments?: Array       // 첨부 파일 (이미지 등)
  timeout?: number          // 타임아웃 (초)
}
```

---

### Phase 3: 에이전트 세션 준비

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **3-1** | 설정 로드 및 워크스페이스/에이전트 ID 해석 | `src/commands/agent.ts` |
| **3-2** | 모델/프로바이더 해석 (Anthropic, OpenAI, Google 등) | `pi-embedded-runner/model.ts` |
| **3-3** | 세션 매니저 오픈 (디스크에서 대화 기록 로드) | `run/attempt.ts:585` |
| **3-4** | 히스토리 정제 (유효성 검사, 고아 tool_result 정리, 턴 수 제한) | `run/attempt.ts:810-841` |
| **3-5** | 시스템 프롬프트 구성 (정체성 + 도구 + 스킬 + 컨텍스트) | `src/agents/system-prompt.ts` |
| **3-6** | 도구 목록 생성 및 등록 | `src/agents/pi-tools.ts` |
| **3-7** | 프롬프트 내 이미지 탐지 및 로드 | `run/images.ts` |

---

### Phase 4: LLM 호출 및 Agentic Loop

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **4-1** | `createAgentSession()`으로 Pi SDK 에이전트 인스턴스 생성 | `run/attempt.ts:671` |
| **4-2** | `streamSimple()`로 LLM API 스트리밍 호출 | `@mariozechner/pi-ai` |
| **4-3** | LLM 응답에서 `text_delta` → 사용자에게 스트리밍 | `pi-embedded-subscribe.ts` |
| **4-4** | `tool_use` 블록 감지 → 도구 실행 → `tool_result` 추가 | SDK 내부 루프 |
| **4-5** | `stop_reason === "end_turn"`까지 4-2~4-4 반복 | SDK 내부 루프 |
| **4-6** | 최종 텍스트 조합 및 세션에 저장 | `run/attempt.ts` |

**LLM API 호출 페이로드:**
```typescript
{
  model: "claude-opus-4-6",       // 설정된 모델
  system: "시스템 프롬프트 전문",     // buildAgentSystemPrompt() 결과
  messages: [                      // 대화 히스토리 + 현재 메시지
    { role: "user", content: "이전 질문" },
    { role: "assistant", content: "이전 답변" },
    { role: "user", content: "현재 질문" }
  ],
  tools: [                         // 사용 가능한 도구 목록
    { name: "exec", description: "...", input_schema: {...} },
    { name: "read", description: "...", input_schema: {...} },
    // ...
  ],
  stream: true,
  max_tokens: 8192
}
```

---

### Phase 5: 응답 스트리밍 및 처리

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **5-1** | `subscribeEmbeddedPiSession()`으로 이벤트 핸들러 등록 | `pi-embedded-subscribe.ts:34` |
| **5-2** | `message_start` → 상태 초기화 | `handlers.messages.ts` |
| **5-3** | `message_update` → 텍스트 청크 축적, `<think>` 태그 제거 | `handlers.messages.ts` |
| **5-4** | `onPartialReply()` → 실시간 스트리밍 출력 (웹/CLI) | `pi-embedded-subscribe.ts` |
| **5-5** | `onBlockReply()` → 단락 단위 소프트 청킹 (메시징 채널) | `EmbeddedBlockChunker` |
| **5-6** | `message_end` → 최종 텍스트 확정 및 블록 플러시 | `handlers.messages.ts` |

**이벤트 핸들러 매핑:**
```
SDK Event                    → Handler Function
─────────────────────────────────────────────────
message_start               → handleMessageStart()
message_update (text)       → handleMessageUpdate()
message_end                 → handleMessageEnd()
tool_execution_start        → handleToolExecutionStart()
tool_execution_update       → handleToolExecutionUpdate()
tool_execution_end          → handleToolExecutionEnd()
agent_start                 → handleAgentStart()
auto_compaction_start/end   → handleAutoCompaction*()
agent_end                   → handleAgentEnd()
```

---

### Phase 6: 응답 전달

| 단계 | 설명 | 핵심 파일 |
|------|------|----------|
| **6-1** | `buildEmbeddedRunPayloads()`로 최종 페이로드 조합 | `run/payloads.ts` |
| **6-2** | `deliverAgentCommandResult()`로 전달 계획 실행 | `commands/agent/delivery.ts` |
| **6-3** | `deliverOutboundPayloads()`로 채널별 포맷 변환 및 전송 | `src/infra/outbound/deliver.ts` |
| **6-4** | 채널 API 호출 (Telegram `sendMessage`, Discord `send` 등) | 각 채널 모듈 |

**채널별 전달 방식:**
| 채널 | 포맷 | API |
|------|------|-----|
| Telegram | HTML | `bot.api.sendMessage()` |
| Discord | Markdown | `client.channels.cache.get().send()` |
| Slack | mrkdwn | `client.chat.postMessage()` |
| Web | Markdown (스트리밍) | WebSocket delta |
| Signal | Plain text | Signal CLI |
| iMessage | Plain text | Mac app bridge |

---

## Agentic Loop 상세

### 이중 루프 구조

OpenClaw Agent는 **이중 루프(Dual Loop)** 구조로 동작한다:

```mermaid
flowchart TD
    Start([사용자 메시지 수신]) --> OuterLoop

    subgraph OuterLoop["외부 루프 (Auth/Error Retry)"]
        direction TB
        OL1[인증 프로필 선택] --> OL2[runEmbeddedAttempt 호출]

        subgraph InnerLoop["내부 루프 (Tool-Use Loop)"]
            direction TB
            IL1[시스템 프롬프트 + 도구 + 히스토리 조합] --> IL2[LLM API 호출]
            IL2 --> IL3{응답 분석}
            IL3 -->|text + tool_use| IL4[도구 실행]
            IL4 --> IL5[tool_result를 메시지에 추가]
            IL5 --> IL2
            IL3 -->|text only<br/>stop_reason: end_turn| IL6[최종 응답 확정]
        end

        OL2 --> InnerLoop
        InnerLoop --> OL3{결과 판정}
        OL3 -->|성공| OL4[페이로드 조합 및 반환]
        OL3 -->|컨텍스트 오버플로우| OL5[세션 컴팩션]
        OL3 -->|인증/과금 오류| OL6[다음 프로필 전환]
        OL3 -->|타임아웃| OL7[프로필 교체]
        OL5 --> OL1
        OL6 --> OL1
        OL7 --> OL1
    end

    OL4 --> Deliver[응답 전달]
    Deliver --> End([사용자에게 응답 수신])
```

### 외부 루프 (run.ts, line 533-1153)

- **목적:** 인증 프로필 로테이션, 에러 복구, 컨텍스트 오버플로우 처리
- **최대 반복:** `MAX_RUN_LOOP_ITERATIONS = 10`
- **중단 조건:** 성공 또는 복구 불가능한 에러

```
while (true) {
  1. 반복 횟수 체크 (최대 10회)
  2. runEmbeddedAttempt() 호출
  3. 결과 판정:
     - 성공 → 페이로드 구성 후 반환
     - 컨텍스트 오버플로우 → compactEmbeddedPiSession() 후 재시도
     - 인증 오류 → 다음 auth profile로 전환 후 재시도
     - 과금 오류 → 프로필 전환 후 재시도
     - 레이트 리밋 → 쿨다운 후 재시도
     - 기타 오류 → 에러 응답 반환
}
```

### 내부 루프 (SDK의 streamSimple 내부)

- **목적:** LLM과 도구 실행의 반복적 상호작용
- **중단 조건:** LLM의 `stop_reason`이 `"tool_use"`가 아닌 경우 (예: `"end_turn"`)

```
loop {
  1. LLM API 호출 (대화 히스토리 + 시스템 프롬프트 + 도구 정의)
  2. 스트리밍 응답 수신 (text_delta 이벤트)
  3. tool_use 블록이 있으면:
     a. 도구 실행 (read, write, exec 등)
     b. 실행 결과를 tool_result 메시지로 추가
     c. 루프 계속 (LLM에 결과 전달)
  4. stop_reason이 "end_turn"이면 루프 종료
}
```

---

## 시스템 프롬프트 구성

시스템 프롬프트는 에이전트가 LLM에 보내는 **첫 번째 지시사항**이다. 에이전트의 정체성, 사용 가능한 도구, 안전 규칙, 워크스페이스 컨텍스트 등을 하나의 텍스트로 조합한다.

### 프롬프트 구성 전체 흐름

```mermaid
sequenceDiagram
    participant Attempt as runEmbeddedAttempt()
    participant BootstrapFiles as bootstrap-files.ts
    participant Workspace as workspace.ts
    participant Hooks as bootstrap-hooks.ts
    participant BuildCtx as buildBootstrapContextFiles()
    participant BuildPrompt as buildEmbeddedSystemPrompt()
    participant CorePrompt as buildAgentSystemPrompt()
    participant Session as Agent Session

    Note over Attempt: 1단계: 워크스페이스 부트스트랩 파일 로드

    Attempt->>BootstrapFiles: resolveBootstrapContextForRun(workspaceDir, config)
    BootstrapFiles->>Workspace: loadWorkspaceBootstrapFiles(workspaceDir)

    Note over Workspace: AGENTS.md, SOUL.md, TOOLS.md,<br/>IDENTITY.md, USER.md, MEMORY.md,<br/>HEARTBEAT.md, BOOTSTRAP.md 탐색

    Workspace-->>BootstrapFiles: WorkspaceBootstrapFile[] (raw files)
    BootstrapFiles->>BootstrapFiles: filterBootstrapFilesForSession(sessionKey)
    BootstrapFiles->>Hooks: applyBootstrapHookOverrides(files, config)

    Note over Hooks: 사용자 정의 훅으로<br/>파일 내용 변환/추가/제거 가능

    Hooks-->>BootstrapFiles: WorkspaceBootstrapFile[] (hooked)
    BootstrapFiles->>BuildCtx: buildBootstrapContextFiles(files, maxChars)

    Note over BuildCtx: 파일별 최대 20,000자 제한<br/>전체 합계 최대 150,000자 제한<br/>초과 시 head(70%) + tail(20%) 트렁케이션

    BuildCtx-->>Attempt: EmbeddedContextFile[] (contextFiles)

    Note over Attempt: 2단계: 시스템 프롬프트 텍스트 조립

    Attempt->>BuildPrompt: buildEmbeddedSystemPrompt(params)
    BuildPrompt->>CorePrompt: buildAgentSystemPrompt(params)

    Note over CorePrompt: 16+ 섹션을 순서대로 조립<br/>(아래 상세 다이어그램 참조)

    CorePrompt-->>BuildPrompt: systemPromptText (string)
    BuildPrompt-->>Attempt: systemPromptText

    Note over Attempt: 3단계: 세션에 적용

    Attempt->>Session: createSystemPromptOverride(systemPromptText)
    Attempt->>Session: applySystemPromptOverrideToSession(session, override)

    Note over Session: session.agent.setSystemPrompt(prompt)<br/>이후 모든 LLM 호출에 이 프롬프트 사용
```

### 프롬프트 섹션 조립 순서

`buildAgentSystemPrompt()` (`src/agents/system-prompt.ts:196`)에서 아래 섹션들을 **순서대로** 조합한다:

```mermaid
flowchart TD
    Start["You are a personal assistant running inside OpenClaw."] --> S1

    subgraph Core["핵심 섹션 (항상 포함)"]
        S1["## Tooling<br/>사용 가능한 도구 목록 + 설명<br/><i>read, write, edit, exec, grep, find, ...</i>"]
        S2["## Tool Call Style<br/>도구 호출 시 설명 여부 규칙"]
        S3["## Safety<br/>안전 규칙 (자기보존 금지, 감독 우선 등)"]
    end

    subgraph Full["Full 모드 전용 섹션"]
        S4["## OpenClaw CLI Quick Reference<br/>CLI 명령어 가이드"]
        S5["## Skills (mandatory)<br/>사용 가능한 스킬 목록 + 선택 규칙"]
        S6["## Memory Recall<br/>메모리 검색/인출 지침"]
        S7["## OpenClaw Self-Update<br/>자가 업데이트 규칙"]
        S8["## Model Aliases<br/>모델 별칭 목록"]
        S9["## Documentation<br/>문서 경로 + 커뮤니티 링크"]
    end

    subgraph Context["컨텍스트 섹션"]
        S10["## Workspace<br/>작업 디렉토리 경로 + 안내"]
        S11["## Sandbox<br/>(샌드박스 활성 시) 컨테이너 경로 + 제약"]
        S12["## Authorized Senders<br/>허용된 발신자 목록"]
        S13["## Current Date & Time<br/>타임존 정보"]
    end

    subgraph Channel["채널/메시징 섹션"]
        S14["## Workspace Files (injected)<br/>프로젝트 컨텍스트 파일 안내"]
        S15["## Reply Tags<br/>[[reply_to_current]] 등 답장 태그 문법"]
        S16["## Messaging<br/>세션 간 메시징, 서브에이전트, message 도구 사용법"]
        S17["## Voice (TTS)<br/>음성 합성 힌트 (설정 시)"]
    end

    subgraph Extra["추가 섹션"]
        S18["## Group Chat Context / Subagent Context<br/>extraSystemPrompt (사용자 지정 추가 프롬프트)"]
        S19["## Reactions<br/>리액션 가이드 (minimal/extensive)"]
        S20["## Reasoning Format<br/>&lt;think&gt;...&lt;/think&gt; + &lt;final&gt;...&lt;/final&gt; 포맷 강제"]
    end

    subgraph Project["프로젝트 컨텍스트"]
        S21["# Project Context<br/>AGENTS.md, SOUL.md, TOOLS.md, MEMORY.md 등<br/>워크스페이스 부트스트랩 파일 내용 주입"]
    end

    subgraph Tail["후미 섹션"]
        S22["## Silent Replies<br/>무응답 시 ⁂ 토큰 사용 규칙"]
        S23["## Heartbeats<br/>하트비트 폴링 응답 규칙"]
        S24["## Runtime<br/>agent=, host=, os=, model=, channel=, thinking= 등"]
    end

    S1 --> S2 --> S3
    S3 --> S4 --> S5 --> S6 --> S7 --> S8 --> S9
    S9 --> S10 --> S11 --> S12 --> S13
    S13 --> S14 --> S15 --> S16 --> S17
    S17 --> S18 --> S19 --> S20
    S20 --> S21
    S21 --> S22 --> S23 --> S24
```

### 프롬프트 모드별 포함 범위

| 모드 | 용도 | 포함 섹션 |
|------|------|----------|
| `"full"` | 메인 에이전트 | 전체 24개 섹션 |
| `"minimal"` | 서브에이전트 | Tooling, Tool Call Style, Safety, Workspace, Runtime만 |
| `"none"` | 최소 정체성 | `"You are a personal assistant running inside OpenClaw."` 한 줄 |

### 각 섹션 상세 설명

#### 1. Identity (정체성)
```
You are a personal assistant running inside OpenClaw.
```
- 모든 모드에서 포함되는 기본 정체성 선언

#### 2. Tooling (도구 목록)
```
## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
- exec: Run shell commands (pty available for TTY-required CLIs)
- grep: Search file contents for patterns
- find: Find files by glob pattern
- message: Send messages and channel actions
- cron: Manage cron jobs and wake events
- sessions_send: Send a message to another session/sub-agent
...
```
- **구성 방식:** `params.toolNames`에서 실제 등록된 도구만 필터링
- **순서:** `toolOrder` 배열에 정의된 순서 → 미등록 도구는 알파벳순 후미
- **설명:** `coreToolSummaries` 맵에서 기본 설명 제공, `params.toolSummaries`로 외부 도구 설명 추가
- **핵심 코드:** `system-prompt.ts:241-334`

#### 3. Tool Call Style (도구 호출 스타일)
```
## Tool Call Style
Default: do not narrate routine, low-risk tool calls (just call the tool).
Narrate only when it helps: multi-step work, complex/challenging problems, sensitive actions.
```

#### 4. Safety (안전 규칙)
```
## Safety
You have no independent goals: do not pursue self-preservation, replication, resource acquisition...
Prioritize safety and human oversight over completion...
Do not manipulate or persuade anyone to expand access or disable safeguards...
```
- Anthropic의 AI 헌법(Constitution)에서 영감을 받은 규칙

#### 5. Skills (스킬)
```
## Skills (mandatory)
Before replying: scan <available_skills> <description> entries.
- If exactly one skill clearly applies: read its SKILL.md at <location> with `read`, then follow it.
- If multiple could apply: choose the most specific one, then read/follow it.
- If none clearly apply: do not read any SKILL.md.
<available_skills>
  <skill><name>commit</name><description>...</description><location>...</location></skill>
  ...
</available_skills>
```
- **동적 생성:** 워크스페이스의 `.agents/skills/` 디렉토리에서 스캔
- **사용 패턴:** LLM이 쿼리를 분석 → 적합한 스킬 선택 → SKILL.md 읽기 → 지시 따르기

#### 6. Memory Recall (메모리)
```
## Memory Recall
Before answering anything about prior work, decisions, dates, people, preferences, or todos:
run memory_search on MEMORY.md + memory/*.md; then use memory_get to pull only the needed lines.
```
- `memory_search`/`memory_get` 도구가 등록된 경우에만 포함
- `citationsMode`에 따라 출처 표시 여부 결정

#### 7. Workspace (워크스페이스)
```
## Workspace
Your working directory is: /Users/user/.openclaw/workspace
Treat this directory as the single global workspace for file operations.
```
- 샌드박스 모드일 경우 컨테이너 경로와 호스트 경로를 구분하여 안내

#### 8. Authorized Senders (허용 발신자)
```
## Authorized Senders
Authorized senders: +821012345678. These senders are allowlisted; do not assume they are the owner.
```
- `ownerDisplay === "hash"`이면 전화번호를 HMAC-SHA256 해시로 변환 (개인정보 보호)

#### 9. Project Context (프로젝트 컨텍스트) - 핵심 섹션
```
# Project Context
The following project context files have been loaded:
If SOUL.md is present, embody its persona and tone.

## AGENTS.md
(AGENTS.md 파일 내용)

## SOUL.md
(SOUL.md 파일 내용 - 에이전트 페르소나)

## TOOLS.md
(TOOLS.md 파일 내용 - 도구 사용 가이드)

## MEMORY.md
(MEMORY.md 파일 내용 - 누적 메모리)
```
- **부트스트랩 파일 로딩 흐름:**

```mermaid
flowchart TD
    A["워크스페이스 디렉토리 스캔"] --> B{파일 존재 여부 체크}

    B -->|존재| C["파일 읽기 (mtime 캐시)"]
    B -->|부재| D["[MISSING] 마커 생성"]

    C --> E["stripFrontMatter()<br/>YAML 프론트매터 제거"]
    E --> F["filterBootstrapFilesForSession()<br/>세션별 필터링"]
    F --> G["applyBootstrapHookOverrides()<br/>사용자 훅으로 변환"]

    G --> H["buildBootstrapContextFiles()"]

    H --> I{파일 크기 체크}
    I -->|≤ 20,000자| J["전체 내용 사용"]
    I -->|> 20,000자| K["트렁케이션"]

    K --> K1["head: 앞 70% 유지"]
    K --> K2["tail: 뒤 20% 유지"]
    K --> K3["중간에 [...truncated...] 마커"]

    J --> L{전체 합계 체크}
    K1 & K2 & K3 --> L
    L -->|≤ 150,000자| M["EmbeddedContextFile[]에 추가"]
    L -->|> 150,000자| N["이후 파일 생략"]

    M --> O["시스템 프롬프트의<br/># Project Context 섹션에 주입"]
```

**부트스트랩 파일 우선순위 및 역할:**

| 파일명 | 역할 | 기본 위치 |
|--------|------|----------|
| `AGENTS.md` | 에이전트 지시사항 (CLAUDE.md와 동일) | `~/.openclaw/workspace/AGENTS.md` |
| `SOUL.md` | 에이전트 페르소나/톤 정의 | `~/.openclaw/workspace/SOUL.md` |
| `TOOLS.md` | 외부 도구 사용 가이드 | `~/.openclaw/workspace/TOOLS.md` |
| `IDENTITY.md` | 에이전트 정체성 설정 | `~/.openclaw/workspace/IDENTITY.md` |
| `USER.md` | 사용자 프로필 정보 | `~/.openclaw/workspace/USER.md` |
| `MEMORY.md` | 누적 메모리 (대화 간 유지) | `~/.openclaw/workspace/MEMORY.md` |
| `HEARTBEAT.md` | 하트비트 프롬프트 커스텀 | `~/.openclaw/workspace/HEARTBEAT.md` |
| `BOOTSTRAP.md` | 추가 부트스트랩 지시사항 | `~/.openclaw/workspace/BOOTSTRAP.md` |

**트렁케이션 전략:**
- 파일당 기본 최대 `20,000`자 (`DEFAULT_BOOTSTRAP_MAX_CHARS`)
- 전체 파일 합계 기본 최대 `150,000`자 (`DEFAULT_BOOTSTRAP_TOTAL_MAX_CHARS`)
- 초과 시: 앞 70% + `[...truncated...]` 마커 + 뒤 20% 보존
- 설정으로 변경 가능: `config.agents.defaults.bootstrapMaxChars` / `bootstrapTotalMaxChars`

#### 10. Silent Replies (무응답 처리)
```
## Silent Replies
When you have nothing to say, respond with ONLY: ⁂
⚠️ Rules:
- It must be your ENTIRE message — nothing else
- Never append it to an actual response
```

#### 11. Heartbeats (하트비트)
```
## Heartbeats
Heartbeat prompt: (configured)
If you receive a heartbeat poll, and there is nothing that needs attention, reply exactly:
HEARTBEAT_OK
```

#### 12. Runtime (런타임 정보)
```
## Runtime
Runtime: agent=default | host=macbook | os=darwin (arm64) | node=v22.x | model=claude-opus-4-6 | channel=telegram | thinking=off
Reasoning: off (hidden unless on/stream).
```

### 최종 시스템 프롬프트 예시 (축약)

```
You are a personal assistant running inside OpenClaw.

## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
- exec: Run shell commands
- grep: Search file contents for patterns
- message: Send messages and channel actions
- cron: Manage cron jobs and wake events
- sessions_send: Send a message to another session
...

## Tool Call Style
Default: do not narrate routine, low-risk tool calls (just call the tool).
...

## Safety
You have no independent goals...
...

## Skills (mandatory)
Before replying: scan <available_skills>...
<available_skills>
  <skill><name>commit</name>...</skill>
  <skill><name>code-review</name>...</skill>
</available_skills>

## Memory Recall
Before answering anything about prior work...

## Workspace
Your working directory is: /Users/user/.openclaw/workspace

## Authorized Senders
Authorized senders: +821012345678.

## Reply Tags
[[reply_to_current]] replies to the triggering message.

## Messaging
- Reply in current session → automatically routes to the source channel
- Cross-session messaging → use sessions_send(...)
...

# Project Context
The following project context files have been loaded:

## AGENTS.md
(에이전트 지시사항 내용)

## SOUL.md
(페르소나 설정 내용)

## MEMORY.md
(누적 메모리 내용)

## Silent Replies
When you have nothing to say, respond with ONLY: ⁂

## Heartbeats
If you receive a heartbeat poll... reply exactly: HEARTBEAT_OK

## Runtime
Runtime: agent=default | host=macbook | model=claude-opus-4-6 | channel=telegram | thinking=off
```

### 프롬프트 구성 핵심 코드 참조

| 컴포넌트 | 파일 | 함수 |
|----------|------|------|
| **메인 프롬프트 조립** | `src/agents/system-prompt.ts:196` | `buildAgentSystemPrompt()` |
| **임베디드 래퍼** | `src/agents/pi-embedded-runner/system-prompt.ts:11` | `buildEmbeddedSystemPrompt()` |
| **세션 적용** | `src/agents/pi-embedded-runner/system-prompt.ts:91` | `applySystemPromptOverrideToSession()` |
| **부트스트랩 파일 해석** | `src/agents/bootstrap-files.ts:72` | `resolveBootstrapContextForRun()` |
| **컨텍스트 파일 빌드** | `src/agents/pi-embedded-helpers/bootstrap.ts:187` | `buildBootstrapContextFiles()` |
| **파일 트렁케이션** | `src/agents/pi-embedded-helpers/bootstrap.ts:114` | `trimBootstrapContent()` |
| **워크스페이스 파일 로드** | `src/agents/workspace.ts` | `loadWorkspaceBootstrapFiles()` |
| **훅 오버라이드** | `src/agents/bootstrap-hooks.ts` | `applyBootstrapHookOverrides()` |
| **런타임 파라미터** | `src/agents/system-prompt-params.ts:35` | `buildSystemPromptParams()` |
| **프롬프트 리포트** | `src/agents/system-prompt-report.ts:120` | `buildSystemPromptReport()` |

---

## 도구(Tool) 실행 파이프라인

### LLM-도구 연결 아키텍처

LLM과 도구가 연결되는 전체 흐름은 **도구 생성 → SDK 포맷 변환 → LLM 세션 전달 → 실행 루프 → 결과 피드백**의 5단계로 구성된다.

```
도구 정의                SDK 포맷 변환              LLM 세션 생성
(pi-tools.ts)      →  (pi-tool-definition-   →  (attempt.ts
(openclaw-tools.ts)     adapter.ts)               → createAgentSession)
                                                        │
                                                        ▼
결과 처리                    이벤트 디스패치          에이전트 루프
(handlers.tools.ts)  ←  (handlers.ts)        ←  (pi-embedded-subscribe.ts)
        │
        ▼
  tool_result로 LLM에 재전달 (루프 반복)
```

#### 1단계: 도구 생성 및 등록

| 파일 | 함수 | 역할 |
|------|------|------|
| `src/agents/pi-tools.ts` | `createOpenClawCodingTools()` | 전체 도구 배열 조립. 정책(policy) 파이프라인으로 필터링, before/after 훅 래핑 후 `AnyAgentTool[]` 반환 |
| `src/agents/openclaw-tools.ts` | `createOpenClawTools()` | OpenClaw 고유 도구 생성 (browser, canvas, message, web search, sessions, subagents, TTS 등) |

#### 2단계: SDK 포맷 변환 (도구 → ToolDefinition)

`src/agents/pi-tool-definition-adapter.ts`에서 내부 `AgentTool[]`을 SDK가 이해하는 `ToolDefinition[]`로 변환한다.

- **`toToolDefinitions()`** — SDK 내장 도구 변환. 각 도구의 `execute` 함수를 before/after 훅으로 감싸서 실행 전후 검증/로깅 수행
- **`toClientToolDefinitions()`** — 클라이언트(호스티드) 도구 변환

```typescript
// 변환 시 래핑되는 실행 흐름 (개념)
execute = async (input) => {
  await beforeToolCallHook(tool, input)    // 실행 전 훅
  const result = await tool.execute(input) // 실제 도구 실행
  await afterToolCallHook(tool, result)    // 실행 후 훅
  return result
}
```

#### 3단계: LLM 세션에 도구 전달

`src/agents/pi-embedded-runner/run/attempt.ts`의 `runEmbeddedAttempt()`에서:

1. `createOpenClawCodingTools()`로 전체 도구 생성
2. `splitSdkTools()`(`src/agents/pi-embedded-runner/tool-split.ts`)로 SDK 내장 도구와 커스텀 도구 분리
3. `createAgentSession()`에 두 종류의 도구를 전달하여 LLM 세션 생성

```typescript
createAgentSession({
  tools: builtInTools,          // SDK 도구 (read, write, edit, exec 등)
  customTools: allCustomTools,  // OpenClaw 도구 + 클라이언트 도구
  sessionManager,
  // ...
})
```

#### 4단계: 에이전트 루프 및 이벤트 디스패치

| 파일 | 역할 |
|------|------|
| `src/agents/pi-embedded-subscribe.ts` | 메인 이벤트 구독 루프. 스트리밍 응답 처리 및 에이전트 루프 생명주기 관리 |
| `src/agents/pi-embedded-subscribe.handlers.ts` | 이벤트 라우터. `tool_execution_start/update/end` 이벤트를 도구 핸들러로 디스패치 |

#### 5단계: 도구 실행 결과 처리 및 LLM 피드백

`src/agents/pi-embedded-subscribe.handlers.tools.ts`에서 도구 실행 이벤트를 처리한다:

| 핸들러 | 역할 |
|--------|------|
| `handleToolExecutionStart()` | 도구 호출 메타데이터 추적, 이벤트 emit |
| `handleToolExecutionUpdate()` | 스트리밍/부분 결과 처리 |
| `handleToolExecutionEnd()` | 결과 검증 및 sanitize, 미디어 URL 추출, 중복 메시지 감지 후 `tool_result`를 대화 히스토리에 추가 → LLM 재호출 |

### 도구 등록 흐름

```mermaid
flowchart TD
    A[createOpenClawCodingTools] --> B[기본 SDK 도구]
    A --> C[실행 도구]
    A --> D[OpenClaw 커스텀 도구]

    B --> B1[createReadTool - 파일 읽기]
    B --> B2[createWriteTool - 파일 쓰기]
    B --> B3[createEditTool - 파일 편집]

    C --> C1[createExecTool - 셸 명령 실행]
    C --> C2[createProcessTool - 백그라운드 프로세스]
    C --> C3[createApplyPatchTool - 코드 패치]

    D --> D1[메시징 도구 - message, sessions_send]
    D --> D2[메모리 도구 - memory_search, memory_get]
    D --> D3[서브에이전트 도구]
    D --> D4[크론 도구]
    D --> D5[채널 도킹 도구]

    B1 & B2 & B3 --> E[wrapToolWorkspaceRootGuard<br/>워크스페이스 경계 보호]
    E --> F[wrapToolWithAbortSignal<br/>중단 시그널 지원]
    F --> G[wrapToolWithBeforeToolCallHook<br/>플러그인 훅]
    G --> H[applyToolPolicyPipeline<br/>보안 정책 적용]
    H --> I[splitSdkTools<br/>SDK 내장 vs 커스텀 분리]
    I --> J[sanitizeToolsForGoogle<br/>프로바이더별 스키마 조정]
    J --> K[최종 도구 목록 → Agent Session에 등록]
```

### 도구 실행 수명주기

```
1. LLM 응답에서 tool_use 블록 감지
   { "type": "tool_use", "name": "exec", "input": { "command": "ls -la" } }

2. SDK가 도구 레지스트리에서 해당 도구 검색

3. 도구 래퍼 체인 실행:
   AbortSignal 체크 → BeforeToolCallHook → PolicyPipeline → 실제 도구 실행

4. 도구 실행 결과 캡처:
   { "type": "tool_result", "content": "total 48\ndrwxr-xr-x..." }

5. 결과 후처리:
   - 결과 크기 초과 시 truncation (Context Guard)
   - 미디어 URL 추출
   - 중복 메시지 감지 (메시징 도구)

6. tool_result 메시지를 대화 히스토리에 추가

7. LLM에 다시 전달 → 다음 판단
```

---

## 세션 및 컨텍스트 관리

### 세션 저장 구조

```
~/.openclaw/
├── sessions/           # 세션 메타데이터
│   └── <sessionId>.json
├── agents/
│   └── <agentId>/
│       └── sessions/
│           └── <sessionId>.jsonl    # 대화 트랜스크립트 (JSONL)
└── credentials/        # 인증 정보
```

### 대화 히스토리 관리 흐름

```mermaid
flowchart TD
    A[세션 파일 로드<br/>SessionManager.open] --> B[히스토리 정제]

    B --> B1[validateAnthropicTurns<br/>Anthropic 포맷 검증]
    B --> B2[validateGeminiTurns<br/>Gemini 포맷 검증]
    B --> B3[sanitizeToolUseResultPairing<br/>고아 tool_result 정리]
    B --> B4[limitHistoryTurns<br/>턴 수 제한]

    B1 & B2 & B3 & B4 --> C[replaceMessages<br/>정제된 히스토리로 교체]

    C --> D[에이전트 실행]

    D --> E{컨텍스트 오버플로우?}
    E -->|예| F[compactEmbeddedPiSession<br/>오래된 메시지 요약]
    E -->|예| G[truncateOversizedToolResults<br/>큰 도구 결과 축소]
    F --> H[축소된 히스토리로 재시도]
    G --> H

    E -->|아니오| I[정상 응답 처리]

    D --> J[auto_compaction<br/>자동 컨텍스트 압축]
    J --> K[SDK가 오래된 대화를 요약하여<br/>토큰 사용량 절감]
```

### 토큰 사용량 추적

```typescript
// 실행 중 누적되는 사용량 정보
{
  inputTokens: number,      // 입력 토큰
  outputTokens: number,     // 출력 토큰
  cacheReadTokens: number,  // 캐시 읽기 토큰
  cacheWriteTokens: number  // 캐시 쓰기 토큰
}
```

---

## 에러 처리 및 Failover

```mermaid
flowchart TD
    A[에러 발생] --> B{에러 분류}

    B -->|인증 오류<br/>isAuthAssistantError| C[다음 Auth 프로필로 전환]
    B -->|과금 오류<br/>isBillingAssistantError| D[다른 프로바이더로 전환]
    B -->|레이트 리밋<br/>isRateLimitAssistantError| E[쿨다운 후 재시도]
    B -->|컨텍스트 오버플로우<br/>isLikelyContextOverflowError| F[세션 컴팩션]
    B -->|타임아웃<br/>isTimeoutErrorMessage| G[프로필 전환 후 재시도]
    B -->|기타 오류| H[에러 응답 반환]

    C --> I{다른 프로필 있음?}
    I -->|예| J[프로필 전환 후 재시도]
    I -->|아니오| K[FailoverError 반환]

    F --> L{컴팩션 횟수 < 3?}
    L -->|예| M[컴팩션 후 재시도]
    L -->|아니오| N[도구 결과 축소 시도]
    N --> O{성공?}
    O -->|예| J
    O -->|아니오| H
```

**에러 분류 함수 (`pi-embedded-helpers.ts`):**
| 함수 | 감지 대상 |
|------|----------|
| `isAuthAssistantError()` | API 키/인증 실패 |
| `isBillingAssistantError()` | 과금 한도 초과 |
| `isRateLimitAssistantError()` | 요청 빈도 제한 |
| `isLikelyContextOverflowError()` | 컨텍스트 윈도우 초과 |
| `isTimeoutErrorMessage()` | 응답 타임아웃 |
| `isFailoverAssistantError()` | 복구 가능한 일반 오류 |

---

## 실전 예시: Discord에서 복합 여행 계획 요청

사용자가 Discord DM에서 아래와 같이 복합적인 요청을 했을 때, 시스템 내부에서 오가는 프롬프트와 메시지를 단계별로 추적한다.

```
두바이 3박5일 여행 계획 짜줘.
최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서
일정표를 파일로 만들어줘
```

이 요청은 **6종의 도구**를 **7회 이상** 호출하는 복합 시나리오다:

| 도구 | 용도 | 호출 횟수 |
|------|------|-----------|
| `web_search` | 항공권, 호텔, 날씨 검색 | 3회 |
| `web_fetch` | 검색 결과 상세 페이지 확인 | 2회 |
| `exec` | 디렉토리 확인 | 1회 |
| `write` | 일정표 Markdown 파일 생성 | 1회 |

### 전체 시퀀스 (프롬프트 흐름 포함)

```mermaid
sequenceDiagram
    actor User as 사용자 (Discord)
    participant Discord as Discord Gateway
    participant Process as processDiscordMessage()
    participant Agent as agentCommand()
    participant Attempt as runEmbeddedAttempt()
    participant LLM as Claude API
    participant WS as web_search
    participant WF as web_fetch
    participant Exec as exec
    participant Write as write
    participant Chunker as Block Chunker
    participant Send as sendMessageDiscord()

    User->>Discord: "두바이 3박5일 여행 계획 짜줘..."
    Discord->>Process: MESSAGE_CREATE
    Process->>Agent: FinalizedMsgContext
    Agent->>Attempt: runEmbeddedAttempt()

    rect rgb(240, 248, 255)
        Note over Attempt,LLM: ── Round 1: 계획 수립 + 항공권 검색 ──
        Attempt->>LLM: system + [user] 복합 요청 + tools
        LLM-->>Attempt: text + tool_use(web_search) x2 병렬
        Note over LLM: "여행 계획을 세우겠습니다. 먼저 항공권과<br/>날씨를 동시에 확인하겠습니다."<br/>+ web_search("서울 두바이 항공권 최저가")<br/>+ web_search("두바이 3월 날씨 여행")
    end

    par 병렬 도구 실행
        Attempt->>WS: web_search("서울 두바이 항공권...")
        WS-->>Attempt: 항공권 검색 결과 5건
    and
        Attempt->>WS: web_search("두바이 3월 날씨...")
        WS-->>Attempt: 날씨 검색 결과 5건
    end

    rect rgb(240, 248, 255)
        Note over Attempt,LLM: ── Round 2: 항공권 상세 + 호텔 검색 ──
        Attempt->>LLM: 이전 대화 + tool_result x2
        LLM-->>Attempt: tool_use(web_fetch) + tool_use(web_search)
        Note over LLM: web_fetch("https://kr.trip.com/...")<br/>+ web_search("두바이 호텔 3박 추천 2026")
    end

    par 병렬 도구 실행
        Attempt->>WF: web_fetch(항공권 상세)
        WF-->>Attempt: 항공권 상세 Markdown
    and
        Attempt->>WS: web_search("두바이 호텔...")
        WS-->>Attempt: 호텔 검색 결과 5건
    end

    rect rgb(240, 248, 255)
        Note over Attempt,LLM: ── Round 3: 호텔 상세 확인 ──
        Attempt->>LLM: 이전 대화 + tool_result x2
        LLM-->>Attempt: tool_use(web_fetch)
        Note over LLM: web_fetch("https://www.booking.com/...")
    end

    Attempt->>WF: web_fetch(호텔 상세)
    WF-->>Attempt: 호텔 상세 Markdown

    rect rgb(240, 248, 255)
        Note over Attempt,LLM: ── Round 4: 파일 생성 ──
        Attempt->>LLM: 이전 대화 + tool_result
        LLM-->>Attempt: tool_use(exec) + tool_use(write)
        Note over LLM: exec("ls ~/travel-plans")<br/>+ write("dubai-trip.md", 일정표 내용)
    end

    Attempt->>Exec: exec("ls ~/travel-plans")
    Exec-->>Attempt: 디렉토리 목록
    Attempt->>Write: write("~/travel-plans/dubai-2026-03.md")
    Write-->>Attempt: 파일 생성 완료

    rect rgb(240, 248, 255)
        Note over Attempt,LLM: ── Round 5: 최종 응답 생성 ──
        Attempt->>LLM: 이전 대화 + tool_result x2
        LLM-->>Attempt: 최종 요약 텍스트
        Note over LLM: stop_reason: "end_turn"
    end

    Attempt->>Chunker: 최종 응답 스트리밍
    Chunker->>Send: onBlockReply (청크 x3)
    Send->>Discord: POST /channels/{id}/messages (x3)
    Discord-->>User: 최종 응답 수신
```

### Step 0: Discord 메시지 수신

**Discord Gateway 원본 이벤트:**
```json
{
  "t": "MESSAGE_CREATE",
  "d": {
    "id": "1234567890",
    "channel_id": "9876543210",
    "author": { "id": "user123", "username": "홍길동" },
    "content": "두바이 3박5일 여행 계획 짜줘.\n최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서\n일정표를 파일로 만들어줘",
    "timestamp": "2026-03-04T10:00:00.000Z",
    "attachments": []
  }
}
```

**processDiscordMessage()가 생성하는 FinalizedMsgContext:**
```json
{
  "Body": "[2026-03-04 10:00 KST] 홍길동: 두바이 3박5일 여행 계획 짜줘.\n최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서\n일정표를 파일로 만들어줘",
  "BodyForAgent": "두바이 3박5일 여행 계획 짜줘.\n최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서\n일정표를 파일로 만들어줘",
  "From": "discord:user123",
  "SessionKey": "discord:default:user123",
  "Surface": "discord"
}
```

### Step 1: 시스템 프롬프트

`buildAgentSystemPrompt()`가 조합하여 LLM에 전달하는 시스템 프롬프트 (핵심 부분만 발췌):

```
You are a personal assistant running inside OpenClaw.

## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
- exec: Run shell commands (pty available for TTY-required CLIs)
- process: Manage background exec sessions
- web_search: Search the web using Brave Search API. Supports region-specific
  and localized search via country and language parameters.
- web_fetch: Fetch and extract readable content from a URL
- browser: Control web browser
- cron: Manage cron jobs and wake events
- message: Send messages and channel actions
- sessions_send: Send a message to another session/sub-agent
- session_status: Show usage/time/model state
TOOLS.md does not control tool availability; it is user guidance...
For long waits, avoid rapid poll loops: use exec with enough yieldMs...

## Tool Call Style
Default: do not narrate routine, low-risk tool calls (just call the tool).
Narrate only when it helps: multi-step work, complex/challenging problems...

## Safety
You have no independent goals...

## Workspace
Your working directory is: /Users/user/.openclaw/workspace

## Authorized Senders
Authorized senders: a1b2c3d4e5f6.

# Project Context

## AGENTS.md
(에이전트 지시사항)

## SOUL.md
(에이전트 페르소나)

## MEMORY.md
(누적 메모리)

## Runtime
Runtime: agent=default | host=macbook | os=darwin (arm64) | node=v22.12.0 |
model=claude-opus-4-6 | channel=discord | thinking=off
```

### Step 2: Round 1 — 계획 수립 + 병렬 검색 (항공권 & 날씨)

**Anthropic Messages API 페이로드:**
```json
{
  "model": "claude-opus-4-6",
  "max_tokens": 16384,
  "stream": true,
  "system": "(시스템 프롬프트 전문)",
  "messages": [
    {
      "role": "user",
      "content": "두바이 3박5일 여행 계획 짜줘.\n최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서\n일정표를 파일로 만들어줘"
    }
  ],
  "tools": [
    {
      "name": "web_search",
      "description": "Search the web using Brave Search API...",
      "input_schema": {
        "type": "object",
        "properties": {
          "query": { "type": "string" },
          "count": { "type": "number", "minimum": 1, "maximum": 10 },
          "search_lang": { "type": "string" },
          "freshness": { "type": "string" }
        },
        "required": ["query"]
      }
    },
    {
      "name": "web_fetch",
      "description": "Fetch and extract readable content from a URL",
      "input_schema": {
        "type": "object",
        "properties": {
          "url": { "type": "string" },
          "extractMode": { "type": "string" },
          "maxChars": { "type": "number" }
        },
        "required": ["url"]
      }
    },
    {
      "name": "exec",
      "description": "Run shell commands...",
      "input_schema": {
        "type": "object",
        "properties": {
          "command": { "type": "string" }
        },
        "required": ["command"]
      }
    },
    {
      "name": "write",
      "description": "Create or overwrite files",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": { "type": "string" },
          "content": { "type": "string" }
        },
        "required": ["path", "content"]
      }
    },
    {
      "name": "read",
      "description": "Read file contents..."
    }
  ]
}
```

**LLM 응답 (Round 1) — 텍스트 + 도구 2개 병렬 호출:**
```json
{
  "content": [
    {
      "type": "text",
      "text": "두바이 3박 5일 여행 계획을 세워드리겠습니다. 항공권, 호텔, 날씨를 먼저 조사하겠습니다."
    },
    {
      "type": "tool_use",
      "id": "toolu_01_flight",
      "name": "web_search",
      "input": {
        "query": "서울 인천 두바이 항공권 최저가 왕복 2026년 3월",
        "count": 5,
        "search_lang": "ko"
      }
    },
    {
      "type": "tool_use",
      "id": "toolu_01_weather",
      "name": "web_search",
      "input": {
        "query": "두바이 3월 날씨 기온 여행 옷차림 2026",
        "count": 5,
        "search_lang": "ko"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

> LLM은 복합 요청을 분석하여 **2개의 web_search를 동시에 호출**.
> SDK가 이 두 도구를 병렬로 실행한다.

### Step 3: 도구 실행 — web_search x2 (병렬)

**도구 1: 항공권 검색 (`toolu_01_flight`)**
```
GET https://api.search.brave.com/res/v1/web/search
    ?q=서울+인천+두바이+항공권+최저가+왕복+2026년+3월&count=5&search_lang=ko
```

반환:
```json
{
  "query": "서울 인천 두바이 항공권 최저가 왕복 2026년 3월",
  "provider": "brave",
  "count": 5,
  "tookMs": 780,
  "externalContent": { "untrusted": true, "source": "web_search", "wrapped": true },
  "results": [
    {
      "title": "[EXTERNAL_CONTENT source=web_search]서울-두바이 항공권 - 트립닷컴[/EXTERNAL_CONTENT]",
      "url": "https://kr.trip.com/flights/seoul-to-dubai/",
      "description": "[EXTERNAL_CONTENT source=web_search]왕복 최저 ₩385,000 (에티하드, 아부다비 경유)...[/EXTERNAL_CONTENT]"
    },
    {
      "title": "[EXTERNAL_CONTENT source=web_search]두바이 항공권 비교 - 스카이스캐너[/EXTERNAL_CONTENT]",
      "url": "https://www.skyscanner.co.kr/transport/flights/icn/dxb/",
      "description": "[EXTERNAL_CONTENT source=web_search]에미레이트항공 직항 ₩520,000, 카타르항공 ₩402,000...[/EXTERNAL_CONTENT]"
    },
    {
      "title": "[EXTERNAL_CONTENT source=web_search]인천-두바이 저가 항공권 | 네이버[/EXTERNAL_CONTENT]",
      "url": "https://flight.naver.com/flights/international/ICN-DXB",
      "description": "[EXTERNAL_CONTENT source=web_search]3월 최저가 ₩430,000...[/EXTERNAL_CONTENT]"
    }
  ]
}
```

**도구 2: 날씨 검색 (`toolu_01_weather`)**
```
GET https://api.search.brave.com/res/v1/web/search
    ?q=두바이+3월+날씨+기온+여행+옷차림+2026&count=5&search_lang=ko
```

반환:
```json
{
  "query": "두바이 3월 날씨 기온 여행 옷차림 2026",
  "provider": "brave",
  "count": 5,
  "tookMs": 654,
  "externalContent": { "untrusted": true, "source": "web_search", "wrapped": true },
  "results": [
    {
      "title": "[EXTERNAL_CONTENT source=web_search]두바이 3월 날씨 - 여행 가이드[/EXTERNAL_CONTENT]",
      "url": "https://www.weather-guide.com/dubai-march",
      "description": "[EXTERNAL_CONTENT source=web_search]평균 기온 22-32°C, 맑은 날 많음, 자외선 강함, 가벼운 여름옷 + 자외선차단제 필수...[/EXTERNAL_CONTENT]"
    },
    {
      "title": "[EXTERNAL_CONTENT source=web_search]두바이 여행 최적 시기 | 트립어드바이저[/EXTERNAL_CONTENT]",
      "url": "https://www.tripadvisor.co.kr/dubai-best-time",
      "description": "[EXTERNAL_CONTENT source=web_search]3월은 두바이 여행 성수기, 비 거의 없음, 습도 낮음...[/EXTERNAL_CONTENT]"
    }
  ]
}
```

### Step 4: Round 2 — 항공권 상세 + 호텔 검색 (병렬)

**Messages API 전달 (누적된 대화):**
```json
{
  "messages": [
    { "role": "user", "content": "두바이 3박5일 여행 계획 짜줘..." },
    {
      "role": "assistant",
      "content": [
        { "type": "text", "text": "두바이 3박 5일 여행 계획을 세워드리겠습니다..." },
        { "type": "tool_use", "id": "toolu_01_flight", "name": "web_search", "input": { "query": "서울 인천 두바이 항공권..." } },
        { "type": "tool_use", "id": "toolu_01_weather", "name": "web_search", "input": { "query": "두바이 3월 날씨..." } }
      ]
    },
    {
      "role": "user",
      "content": [
        { "type": "tool_result", "tool_use_id": "toolu_01_flight", "content": "{항공권 검색 결과 JSON}" },
        { "type": "tool_result", "tool_use_id": "toolu_01_weather", "content": "{날씨 검색 결과 JSON}" }
      ]
    }
  ]
}
```

**LLM 응답 (Round 2) — 도구 2개 병렬 호출:**
```json
{
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_02_flight_detail",
      "name": "web_fetch",
      "input": {
        "url": "https://kr.trip.com/flights/seoul-to-dubai/",
        "extractMode": "markdown",
        "maxChars": 15000
      }
    },
    {
      "type": "tool_use",
      "id": "toolu_02_hotel",
      "name": "web_search",
      "input": {
        "query": "두바이 호텔 3박 추천 가성비 2026년 3월 다운타운 마리나",
        "count": 5,
        "search_lang": "ko"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

> LLM은 최저가 항공권 상세를 확인하면서, 동시에 호텔 검색도 시작.

### Step 5: 도구 실행 — web_fetch + web_search (병렬)

**도구 3: 항공권 상세 (`toolu_02_flight_detail`)**

```
HTTP GET https://kr.trip.com/flights/seoul-to-dubai/
```
→ Readability.js로 HTML→Markdown 변환 → 보안 래핑

반환:
```json
{
  "url": "https://kr.trip.com/flights/seoul-to-dubai/",
  "status": 200,
  "extractor": "readability",
  "text": "[EXTERNAL_CONTENT source=web_fetch]\n# 서울(ICN) → 두바이(DXB)\n\n| 항공사 | 출발 | 도착 | 경유 | 가격 |\n|--------|------|------|------|------|\n| 에티하드항공 | 03/15 22:30 | 03/16 10:45 | 아부다비 1회 | ₩385,000 |\n| 카타르항공 | 03/15 23:55 | 03/16 12:30 | 도하 1회 | ₩402,000 |\n| 에미레이트항공 | 03/15 23:50 | 03/16 05:20 | 직항 | ₩520,000 |\n| 대한항공 | 03/16 00:30 | 03/16 06:10 | 직항 | ₩589,000 |\n[/EXTERNAL_CONTENT]",
  "tookMs": 1342,
  "externalContent": { "untrusted": true, "source": "web_fetch", "wrapped": true }
}
```

**도구 4: 호텔 검색 (`toolu_02_hotel`)**
```
GET https://api.search.brave.com/res/v1/web/search
    ?q=두바이+호텔+3박+추천+가성비+2026년+3월+다운타운+마리나&count=5&search_lang=ko
```

반환:
```json
{
  "query": "두바이 호텔 3박 추천 가성비 2026년 3월",
  "provider": "brave",
  "count": 5,
  "tookMs": 712,
  "results": [
    {
      "title": "[EXTERNAL_CONTENT source=web_search]두바이 호텔 TOP 10 - Booking.com[/EXTERNAL_CONTENT]",
      "url": "https://www.booking.com/city/ae/dubai.ko.html",
      "description": "[EXTERNAL_CONTENT source=web_search]로브 다운타운 1박 ₩89,000, JW 메리어트 마키 ₩145,000...[/EXTERNAL_CONTENT]"
    },
    {
      "title": "[EXTERNAL_CONTENT source=web_search]두바이 가성비 호텔 추천 - 트립어드바이저[/EXTERNAL_CONTENT]",
      "url": "https://www.tripadvisor.co.kr/Hotels-g295424-Dubai-Hotels.html",
      "description": "[EXTERNAL_CONTENT source=web_search]알바샤 빕 호텔 1박 ₩55,000, 힐튼 가든인 ₩78,000...[/EXTERNAL_CONTENT]"
    },
    {
      "title": "[EXTERNAL_CONTENT source=web_search]2026 두바이 숙소 총정리 - 마이리얼트립[/EXTERNAL_CONTENT]",
      "url": "https://www.myrealtrip.com/offers/dubai-hotels",
      "description": "[EXTERNAL_CONTENT source=web_search]다운타운 지역 평균 1박 ₩100,000~₩150,000, 마리나 지역 ₩70,000~₩120,000...[/EXTERNAL_CONTENT]"
    }
  ]
}
```

### Step 6: Round 3 — 호텔 상세 확인

**Messages API 전달 (누적 5개 메시지):**
```json
{
  "messages": [
    { "role": "user", "content": "두바이 3박5일 여행 계획 짜줘..." },
    { "role": "assistant", "content": [text + tool_use x2] },
    { "role": "user", "content": [tool_result x2 (항공권+날씨 검색)] },
    { "role": "assistant", "content": [tool_use x2 (항공권상세+호텔검색)] },
    { "role": "user", "content": [tool_result x2 (항공권상세+호텔검색)] }
  ]
}
```

**LLM 응답 (Round 3):**
```json
{
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_03_hotel_detail",
      "name": "web_fetch",
      "input": {
        "url": "https://www.booking.com/city/ae/dubai.ko.html",
        "extractMode": "markdown",
        "maxChars": 15000
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

**도구 5: 호텔 상세 (`toolu_03_hotel_detail`)**
```
HTTP GET https://www.booking.com/city/ae/dubai.ko.html
```

반환:
```json
{
  "url": "https://www.booking.com/city/ae/dubai.ko.html",
  "status": 200,
  "extractor": "readability",
  "text": "[EXTERNAL_CONTENT source=web_fetch]\n# 두바이 인기 호텔\n\n## 가성비 TOP\n| 호텔 | 위치 | 1박 가격 | 평점 |\n|------|------|---------|------|\n| 로브 다운타운 | 다운타운 | ₩89,000 | 8.7 |\n| 힐튼 가든인 알미나 | 버즈만 | ₩78,000 | 8.4 |\n| 알바샤 빕 호텔 | 알바샤 | ₩55,000 | 8.1 |\n\n## 럭셔리\n| 호텔 | 위치 | 1박 가격 | 평점 |\n|------|------|---------|------|\n| 아틀란티스 더 팜 | 팜 주메이라 | ₩350,000 | 9.1 |\n| 버즈 알 아랍 | 주메이라 | ₩1,200,000 | 9.5 |\n[/EXTERNAL_CONTENT]",
  "tookMs": 2103
}
```

### Step 7: Round 4 — 파일 생성 (exec + write)

**Messages API 전달 (누적 7개 메시지):**
```json
{
  "messages": [
    { "role": "user", "content": "두바이 3박5일 여행 계획 짜줘..." },
    { "role": "assistant", "content": [text + tool_use x2] },
    { "role": "user", "content": [tool_result x2] },
    { "role": "assistant", "content": [tool_use x2] },
    { "role": "user", "content": [tool_result x2] },
    { "role": "assistant", "content": [tool_use x1] },
    { "role": "user", "content": [tool_result x1 (호텔 상세)] }
  ]
}
```

**LLM 응답 (Round 4) — exec + write 호출:**
```json
{
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_04_mkdir",
      "name": "exec",
      "input": {
        "command": "mkdir -p ~/travel-plans"
      }
    },
    {
      "type": "tool_use",
      "id": "toolu_04_write",
      "name": "write",
      "input": {
        "path": "/Users/user/travel-plans/dubai-2026-03.md",
        "content": "# 두바이 3박 5일 여행 계획\n\n> 기간: 2026년 3월 15일(일) ~ 3월 19일(목)\n\n## 항공편\n\n| 구간 | 항공사 | 시간 | 가격 |\n|------|--------|------|------|\n| 인천→두바이 | 에티하드항공 | 3/15 22:30→3/16 10:45 (아부다비 경유) | ₩385,000 (왕복) |\n| 두바이→인천 | 에티하드항공 | 3/19 14:00→3/20 05:30 (아부다비 경유) | (왕복 포함) |\n\n## 숙소\n\n**로브 다운타운** (3박)\n- 위치: 다운타운, 버즈칼리파 도보 5분\n- 가격: ₩89,000/박 × 3박 = **₩267,000**\n- 평점: 8.7/10\n\n## 날씨\n\n- 기온: 22~32°C (맑음, 비 거의 없음)\n- 습도: 낮음\n- 준비물: 여름옷, 자외선차단제, 선글라스, 실내 냉방 대비 가디건\n\n## 일정\n\n### Day 1 (3/16 일) - 도착 & 다운타운\n- 10:45 두바이 도착, 호텔 체크인\n- 14:00 두바이 몰 & 수족관\n- 18:00 버즈칼리파 전망대 (At The Top, 사전 예약 필수)\n- 20:00 두바이 분수쇼 감상 + 저녁식사\n\n### Day 2 (3/17 월) - 올드 두바이 & 문화\n- 09:00 아브라(수상택시)로 두바이 크릭 횡단\n- 10:00 골드 수크 & 스파이스 수크\n- 13:00 알파히디 역사지구 점심\n- 15:00 두바이 박물관 (알파히디 요새)\n- 18:00 주메이라 모스크 방문\n- 20:00 마디낫 주메이라 저녁식사\n\n### Day 3 (3/18 화) - 사막 & 마리나\n- 09:00 두바이 마리나 산책 & 브런치\n- 11:00 팜 주메이라 모노레일\n- 13:00 아틀란티스 워터파크 (반나절)\n- 16:00 호텔 휴식\n- 17:00 사막 사파리 투어 (픽업)\n  - 듄 배싱, 낙타 타기, BBQ 디너, 벨리댄스\n- 22:00 호텔 귀환\n\n### Day 4 (3/19 수) - 출발\n- 09:00 체크아웃\n- 10:00 두바이 몰 면세 쇼핑\n- 12:00 공항 이동\n- 14:00 두바이 출발\n\n## 예산 총정리\n\n| 항목 | 비용 |\n|------|------|\n| 항공권 (왕복) | ₩385,000 |\n| 숙소 (3박) | ₩267,000 |\n| 사막 사파리 | ₩80,000 |\n| 버즈칼리파 전망대 | ₩45,000 |\n| 식비 (4일) | ₩200,000 |\n| 교통비 | ₩50,000 |\n| 기타 (입장료 등) | ₩73,000 |\n| **합계** | **₩1,100,000** |\n\n## 예약 링크\n\n- [항공권 - 트립닷컴](https://kr.trip.com/flights/seoul-to-dubai/)\n- [호텔 - Booking.com](https://www.booking.com/city/ae/dubai.ko.html)\n- [사막 사파리 - GetYourGuide](https://www.getyourguide.com/dubai-l173/desert-safari/)\n"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

**도구 6: exec 실행 결과:**
```
(디렉토리 생성됨)
```

**도구 7: write 실행 결과:**
```json
{
  "path": "/Users/user/travel-plans/dubai-2026-03.md",
  "bytesWritten": 2847,
  "created": true
}
```

### Step 8: Round 5 — 최종 응답

**Messages API 전달 (누적 9개 메시지):**
```json
{
  "messages": [
    { "role": "user", "content": "두바이 3박5일 여행 계획 짜줘..." },
    { "role": "assistant", "content": [text + tool_use x2] },
    { "role": "user", "content": [tool_result x2] },
    { "role": "assistant", "content": [tool_use x2] },
    { "role": "user", "content": [tool_result x2] },
    { "role": "assistant", "content": [tool_use x1] },
    { "role": "user", "content": [tool_result x1] },
    { "role": "assistant", "content": [tool_use x2 (exec + write)] },
    { "role": "user", "content": [tool_result x2] }
  ]
}
```

**LLM 최종 응답 (Round 5):**
```json
{
  "content": [
    {
      "type": "text",
      "text": "두바이 3박 5일 여행 계획을 완성했습니다!\n\n**항공편:** 에티하드항공 왕복 ₩385,000 (아부다비 경유)\n**숙소:** 로브 다운타운 3박 ₩267,000 (버즈칼리파 도보 5분)\n**날씨:** 22~32°C, 맑음 (여름옷 + 자외선차단제)\n\n**일정 요약:**\n- Day 1: 도착 → 두바이 몰 → 버즈칼리파 전망대 → 분수쇼\n- Day 2: 올드 두바이 → 골드수크 → 두바이 박물관 → 마디낫 주메이라\n- Day 3: 마리나 → 팜 주메이라 → 사막 사파리(듄배싱+BBQ)\n- Day 4: 면세 쇼핑 → 출발\n\n**총 예산: 약 ₩1,100,000** (항공+숙소+식비+관광)\n\n상세 일정표를 파일로 저장했습니다:\n📄 `~/travel-plans/dubai-2026-03.md`\n\n일정이나 예산 조정이 필요하면 말씀해주세요!"
    }
  ],
  "stop_reason": "end_turn"
}
```

### Step 9: Discord로 응답 전달

**Block Chunker가 텍스트를 Discord 제한(2000자)에 맞게 청킹 후 전송:**

사용자가 Discord에서 보는 메시지들:

> 두바이 3박 5일 여행 계획을 세워드리겠습니다. 항공권, 호텔, 날씨를 먼저 조사하겠습니다.

> 두바이 3박 5일 여행 계획을 완성했습니다!
>
> **항공편:** 에티하드항공 왕복 ₩385,000 (아부다비 경유)
> **숙소:** 로브 다운타운 3박 ₩267,000 (버즈칼리파 도보 5분)
> **날씨:** 22~32°C, 맑음 (여름옷 + 자외선차단제)
>
> **일정 요약:**
> - Day 1: 도착 → 두바이 몰 → 버즈칼리파 전망대 → 분수쇼
> - Day 2: 올드 두바이 → 골드수크 → 두바이 박물관
> - Day 3: 마리나 → 팜 주메이라 → 사막 사파리
> - Day 4: 면세 쇼핑 → 출발
>
> **총 예산: 약 ₩1,100,000**
>
> 상세 일정표를 파일로 저장했습니다:
> `~/travel-plans/dubai-2026-03.md`

### 프롬프트 흐름 요약

```mermaid
flowchart TD
    subgraph Input["사용자 입력"]
        U["두바이 3박5일 여행 계획 짜줘<br/>항공권+호텔+날씨 확인해서 일정표 파일로"]
    end

    subgraph R1["Round 1"]
        direction TB
        R1_IN["[system] 시스템 프롬프트<br/>[user] 복합 요청<br/>[tools] web_search, web_fetch, exec, write, ..."]
        R1_OUT["[assistant] 텍스트 + tool_use x2<br/>web_search(항공권) + web_search(날씨)"]
        R1_IN --> R1_OUT
    end

    subgraph T1["도구 실행 (병렬)"]
        T1a["Brave API: 항공권 5건"]
        T1b["Brave API: 날씨 5건"]
    end

    subgraph R2["Round 2"]
        R2_IN["[이전 대화] + tool_result x2"]
        R2_OUT["[assistant] tool_use x2<br/>web_fetch(항공권 상세) + web_search(호텔)"]
        R2_IN --> R2_OUT
    end

    subgraph T2["도구 실행 (병렬)"]
        T2a["HTTP GET + Readability: 항공권"]
        T2b["Brave API: 호텔 5건"]
    end

    subgraph R3["Round 3"]
        R3_IN["[이전 대화] + tool_result x2"]
        R3_OUT["[assistant] tool_use x1<br/>web_fetch(호텔 상세)"]
        R3_IN --> R3_OUT
    end

    subgraph T3["도구 실행"]
        T3a["HTTP GET + Readability: 호텔"]
    end

    subgraph R4["Round 4"]
        R4_IN["[이전 대화] + tool_result x1"]
        R4_OUT["[assistant] tool_use x2<br/>exec(mkdir) + write(일정표.md)"]
        R4_IN --> R4_OUT
    end

    subgraph T4["도구 실행"]
        T4a["셸: mkdir -p"]
        T4b["파일 생성: dubai-2026-03.md"]
    end

    subgraph R5["Round 5"]
        R5_IN["[이전 대화] + tool_result x2"]
        R5_OUT["[assistant] 최종 요약 텍스트<br/>stop_reason: end_turn"]
        R5_IN --> R5_OUT
    end

    subgraph Output["Discord 전달"]
        D["Block Chunker → Discord API<br/>사용자에게 최종 응답 + 파일 경로"]
    end

    U --> R1 --> T1 --> R2 --> T2 --> R3 --> T3 --> R4 --> T4 --> R5 --> Output
```

### 토큰 사용량 추정

| Round | 입력 토큰 (추정) | 출력 토큰 (추정) | 누적 | 사용 도구 |
|-------|-----------------|-----------------|------|----------|
| **Round 1** | ~3,500 (시스템프롬프트 + 유저 + 스키마) | ~120 (텍스트 + tool_use x2) | 3,620 | web_search x2 |
| **Round 2** | ~5,800 (이전 대화 + tool_result x2) | ~80 (tool_use x2) | 9,500 | web_fetch + web_search |
| **Round 3** | ~9,200 (이전 대화 + tool_result x2) | ~40 (tool_use x1) | 18,740 | web_fetch |
| **Round 4** | ~13,500 (이전 대화 + tool_result x1) | ~1,800 (exec + write 대용량) | 34,040 | exec + write |
| **Round 5** | ~16,000 (이전 대화 + tool_result x2) | ~250 (최종 요약) | **50,290** | (없음) |

> **총 토큰: ~50,000** (단순 검색 대비 ~3배). 복합 요청일수록 대화 누적으로 입력 토큰이 급증.
> 캐시 활용 시 Round 2~5의 시스템 프롬프트 + 이전 대화 부분이 `cacheReadTokens`로 전환.

### 도구별 실행 시간 추정

```
Round 1: web_search x2 (병렬)     ~800ms
Round 2: web_fetch + web_search (병렬) ~1,500ms
Round 3: web_fetch x1              ~2,100ms
Round 4: exec + write (병렬)       ~100ms
LLM 호출 x5 (각 ~2-5초)           ~15,000ms
─────────────────────────────────────────
총 예상 소요시간:                  ~20-25초
```

---

## 쉬운 설명: 복합 여행 계획 요청의 동작 원리

위의 복합 여행 계획 예시를 처음 접하는 사람도 이해할 수 있도록, 등장인물(모듈)과 전체 흐름을 단계별로 풀어 설명한다.

### 이게 뭔데?

사용자(홍길동)가 Discord 채팅방에서 AI 비서(OpenClaw)에게 이렇게 말했다:

```
두바이 3박5일 여행 계획 짜줘.
최저가 항공권이랑 호텔 찾아보고, 현지 날씨도 확인해서
일정표를 파일로 만들어줘
```

이 **한마디** 때문에 AI 비서가 뒤에서 엄청나게 바쁘게 일한다. 마치 심부름꾼이 여러 가게를 돌아다니는 것과 같다.

### 등장인물 소개 (모듈 = 각자 맡은 역할이 있는 일꾼)

| 등장인물 (모듈) | 하는 일 | 비유 |
|----------------|---------|------|
| **사용자 (홍길동)** | Discord에서 메시지를 보내는 사람 | 편의점에 와서 주문하는 손님 |
| **Discord Gateway** | Discord가 메시지를 OpenClaw에 전달 | 편의점 카운터 직원 |
| **processDiscordMessage()** | Discord 메시지를 정리 | 주문서를 깔끔하게 정리하는 사람 |
| **agentCommand()** | AI 비서에게 일을 시키는 관리자 | 팀장 |
| **runEmbeddedAttempt()** | AI와 도구를 실제로 연결해서 돌리는 엔진 | 실제로 일을 하는 사무실 |
| **Claude API (LLM)** | 진짜 AI 두뇌. 생각하고 판단하는 역할 | 천재 비서의 머리 |
| **web_search** | 인터넷 검색 도구 | 네이버/구글 검색해주는 조수 |
| **web_fetch** | 웹페이지를 열어서 읽는 도구 | 홈페이지에 들어가서 자세히 보는 조수 |
| **exec** | 컴퓨터 명령어를 실행하는 도구 | 컴퓨터를 대신 조작해주는 조수 |
| **write** | 파일을 만드는 도구 | 문서를 작성해서 저장해주는 조수 |
| **Block Chunker** | 긴 답변을 Discord 크기에 맞게 자르는 역할 | 긴 편지를 엽서 크기로 나눠주는 사람 |

### 모듈간 관계도 (누가 누구에게 일을 시키나?)

```
사용자 (홍길동)
    |
    | Discord에 메시지를 보냄
    v
+--------------+
|   Discord    | <-- 채팅 앱 (카카오톡 같은 것)
|   Gateway    |
+------+-------+
       | MESSAGE_CREATE 이벤트 전달
       v
+----------------------+
| processDiscordMessage| <-- "누가, 언제, 뭐라고 했는지" 정리
+------+---------------+
       | 정리된 주문서 (FinalizedMsgContext) 전달
       v
+--------------+
| agentCommand | <-- 팀장: "AI야, 이 일 처리해"
+------+-------+
       |
       v
+-----------------------+
|  runEmbeddedAttempt   | <-- 실제 작업실
|                       |
|  +---------+          |
|  | Claude  |<-------->|---- 도구들 ----+
|  |  API    |  생각하고 |               |
|  | (AI 두뇌)|  도구 요청|  +-----------+
|  +---------+          |  |web_search | 인터넷 검색
|                       |  |web_fetch  | 웹페이지 읽기
|       ^               |  |exec       | 컴퓨터 명령
|       | 도구 결과를    |  |write      | 파일 만들기
|       | 다시 AI에게    |  +-----------+
|       +---------------|--------------+
+-----------------------+
       | 최종 답변 완성!
       v
+--------------+
| Block Chunker| <-- 2000자씩 잘라줌 (Discord 제한)
+------+-------+
       |
       v
+--------------+
|   Discord    | --> 사용자에게 답변 전달!
+--------------+
```

### 핵심 개념: "라운드"란?

AI는 한 번에 모든 걸 끝내지 않는다. **여러 번 왔다갔다** 하면서 일을 처리한다:

```
AI에게 질문 --> AI가 생각 --> "도구 써야겠다!" --> 도구 실행 --> 결과를 AI에게 다시 전달
                                                                |
                                         이게 1라운드!  <-------+

AI가 또 생각 --> "또 도구가 필요해!" --> 도구 실행 --> 결과를 AI에게 다시 전달
                                                      |
                                    이게 2라운드! <-----+

... (반복) ...

AI가 생각 --> "이제 다 모았다! 답변 완성!" --> 끝! (end_turn)
                    이게 마지막 라운드!
```

### 단계별 흐름: 피자 배달에 비유하면

#### Step 0: 주문 접수

홍길동이 Discord에서 메시지를 보낸다. Discord가 이것을 JSON 봉투에 담아 전달한다:

```
봉투 내용:
- 보낸 사람: 홍길동 (ID: user123)
- 보낸 곳: Discord DM (채널 ID: 9876543210)
- 내용: "두바이 3박5일 여행 계획 짜줘..."
- 시간: 2026년 3월 4일 오전 10시
```

`processDiscordMessage()`가 이 봉투를 **정리된 주문서(FinalizedMsgContext)**로 바꾼다:

```
주문서:
- Body: 날짜+이름+메시지 (기록용)
- BodyForAgent: 메시지 내용만 (AI가 읽을 용)
- From: discord:user123
- Surface: discord
```

#### Step 1: AI에게 명찰과 도구를 줌 (시스템 프롬프트)

AI가 일을 시작하기 전에, "너는 누구고, 뭘 할 수 있는지" 알려준다. 이것을 **시스템 프롬프트**라고 한다:

```
시스템 프롬프트 = AI의 명찰 + 도구 목록

명찰: "너는 OpenClaw 개인 비서야"
도구 목록:
  - web_search: 인터넷 검색 가능
  - web_fetch: 웹페이지 내용 읽기 가능
  - exec: 컴퓨터 명령어 실행 가능
  - write: 파일 만들기 가능
  - read: 파일 읽기 가능
  - ... (그 외 여러 도구)
```

#### Round 1: "일단 항공권이랑 날씨부터 검색하자!"

AI의 생각: "사용자가 원하는 게 항공권, 호텔, 날씨+일정표 파일이네. 항공권이랑 날씨는 **동시에** 검색할 수 있겠다!"

```
         web_search (1)            web_search (2)
         항공권 검색                날씨 검색
             |                         |
             v                         v
    +----------------+       +----------------+
    | Brave 검색 API |       | Brave 검색 API |
    +--------+-------+       +--------+-------+
             |                         |
             v                         v
    결과: 에티하드 38.5만원    결과: 22~32도, 맑음
          카타르 40.2만원            비 거의 없음
          에미레이트 52만원

    <-- 약 0.8초 소요 (동시에 하니까 빠름!) -->
```

> **병렬 실행**: 양손으로 동시에 전화하는 것처럼, 2개의 검색을 동시에 실행한다.

#### Round 2: "항공권 상세 페이지 열어보고, 호텔도 검색하자!"

AI가 Round 1의 결과를 보고 또 생각한다: "트립닷컴이 가장 싸네. 상세 페이지를 열어서 확인하고, 호텔도 동시에 검색하자!"

```
      web_fetch (3)                web_search (4)
   항공권 상세 페이지 읽기          호텔 검색
          |                            |
          v                            v
  +-------------------+      +----------------+
  | 트립닷컴 웹페이지  |      | Brave 검색 API |
  | HTML을 읽기 쉬운   |      +--------+-------+
  | 형태로 변환        |               |
  +---------+---------+               v
            |                Booking.com 로브호텔 8.9만원
            v                힐튼가든인 7.8만원
  에티하드 22:30출발 38.5만원
  카타르 23:55출발 40.2만원
  에미레이트 직항 52만원
```

#### Round 3: "호텔도 상세 페이지를 확인하자"

```
      web_fetch (5)
   호텔 상세 페이지 읽기
          |
          v
  +-------------------+
  | Booking.com       |
  | 호텔 가격, 평점,  |
  | 위치 정보 추출    |
  +---------+---------+
            |
            v
  로브 다운타운 8.9만원 평점 8.7
  힐튼 가든인 7.8만원 평점 8.4
  아틀란티스 35만원 평점 9.1
```

#### Round 4: "이제 정보가 다 모였다! 파일을 만들자"

AI의 생각: "항공권, 호텔, 날씨 다 모았다! 이제 폴더 만들고 일정표 파일을 쓰자!"

```
      exec (6)                   write (7)
   폴더 만들기                  일정표 파일 생성
       |                            |
       v                            v
  ~/travel-plans/            dubai-2026-03.md
  폴더 생성 완료!             (2,847 바이트 작성)

  파일 내용:
  +----------------------------------+
  | # 두바이 3박 5일 여행 계획        |
  | ## 항공편: 에티하드 왕복 38.5만원  |
  | ## 숙소: 로브 다운타운 3박 26.7만원|
  | ## 날씨: 22~32도, 맑음           |
  | ## 일정                          |
  |  Day 1: 도착, 두바이 몰, 버즈칼리파|
  |  Day 2: 올드 두바이, 골드수크     |
  |  Day 3: 마리나, 사막 사파리       |
  |  Day 4: 면세 쇼핑, 출발          |
  | ## 총 예산: 약 110만원            |
  +----------------------------------+
```

#### Round 5: "최종 답변을 만들자!" (마지막)

AI가 모든 정보를 요약해서 최종 답변을 생성한다. `stop_reason: "end_turn"` — 더 이상 도구를 부를 필요 없이, 여기서 끝!

#### 마지막: Discord로 답변 전달

Discord에는 한 번에 **2000자까지만** 보낼 수 있다. 그래서 Block Chunker가 긴 답변을 잘라서 여러 메시지로 나눠 보낸다:

```
긴 답변 (3000자)
    |
    v
Block Chunker: "2000자 넘네? 나눠야지!"
    |
    +---> 메시지 1: 계획 세워드리겠습니다...
    +---> 메시지 2: 최종 요약 + 파일 경로
    |
    v
Discord API로 각각 전송 --> 사용자가 채팅에서 읽음!
```

### 도구 사용 횟수 요약

| 도구 | 하는 일 | 호출 횟수 |
|------|---------|----------|
| `web_search` | 인터넷에서 검색 (항공권, 날씨, 호텔) | 3번 |
| `web_fetch` | 웹페이지 열어서 자세히 읽기 (항공권 상세, 호텔 상세) | 2번 |
| `exec` | 컴퓨터에게 명령 (폴더 만들기) | 1번 |
| `write` | 파일 만들기 (일정표 마크다운 파일) | 1번 |
| **합계** | | **7번** |

### "누적 대화"란? (비용이 늘어나는 이유)

AI는 **기억이 없다**. 매 라운드마다 이전 대화 전체를 다시 읽어야 한다. 그래서 라운드가 늘어날수록 보내야 하는 데이터(토큰)가 점점 많아진다:

```
Round 1: [사용자 메시지]                              --> 약 3,500 토큰 (짧다!)
Round 2: [사용자 메시지 + AI 대답 + 도구결과 2개]      --> 약 5,800 토큰
Round 3: [위 전부 + AI 대답 + 도구결과 2개]            --> 약 9,200 토큰
Round 4: [위 전부 + AI 대답 + 도구결과 1개]            --> 약 13,500 토큰
Round 5: [위 전부 + AI 대답 + 도구결과 2개]            --> 약 16,000 토큰

총 사용 토큰: 약 50,000개 (책 한 권 분량!)
```

> 이것이 **복합 요청이 단순 검색보다 약 3배 비싼 이유**다. 매번 이전 대화를 전부 다시 보내야 하므로, 라운드가 늘수록 입력 토큰이 급증한다.

### 시간 소요 요약

```
Round 1: AI 생각 ~3초 + 검색 ~0.8초    = 약 4초
Round 2: AI 생각 ~3초 + 검색+읽기 ~1.5초 = 약 4.5초
Round 3: AI 생각 ~3초 + 읽기 ~2.1초     = 약 5초
Round 4: AI 생각 ~4초 + 파일작업 ~0.1초  = 약 4초
Round 5: AI 생각 ~3초                   = 약 3초
---------------------------------------------
총: 약 20~25초
```

> 사용자 홍길동은 Discord에서 약 **20초** 기다리면 완성된 여행 계획을 받아볼 수 있다.

### 한 줄 요약

> 사용자가 Discord에서 한마디 하면 → 메시지 정리 → AI가 5번 생각하면서 7번 도구를 쓰고 → 결과를 모아 일정표 파일을 만들고 → 요약을 Discord로 보내준다. 이 모든 게 약 20초 안에!

---

## 핵심 파일 참조

| 컴포넌트 | 파일 경로 | 핵심 함수 |
|----------|----------|----------|
| **CLI 진입점** | `src/entry.ts` | `main()` |
| **CLI 라우터** | `src/cli/run-main.ts` | `runCli()` |
| **프로그램 빌더** | `src/cli/program/build-program.ts` | `buildProgram()` |
| **에이전트 명령 (게이트웨이)** | `src/commands/agent-via-gateway.ts` | `agentViaGatewayCommand()` |
| **에이전트 명령 (로컬)** | `src/commands/agent.ts` | `agentCommand()` |
| **게이트웨이 에이전트 핸들러** | `src/gateway/server-methods/agent.ts` | `agentHandlers.agent()` |
| **임베디드 Pi 러너** | `src/agents/pi-embedded-runner/run.ts` | `runEmbeddedPiAgent()` |
| **실행 시도** | `src/agents/pi-embedded-runner/run/attempt.ts` | `runEmbeddedAttempt()` |
| **시스템 프롬프트** | `src/agents/system-prompt.ts` | `buildAgentSystemPrompt()` |
| **도구 생성** | `src/agents/pi-tools.ts` | `createOpenClawCodingTools()` |
| **모델 해석** | `src/agents/pi-embedded-runner/model.ts` | `resolveModel()` |
| **스트리밍 구독** | `src/agents/pi-embedded-subscribe.ts` | `subscribeEmbeddedPiSession()` |
| **이벤트 핸들러** | `src/agents/pi-embedded-subscribe.handlers.ts` | `createEmbeddedPiSessionEventHandler()` |
| **메시지 핸들러** | `src/agents/pi-embedded-subscribe.handlers.messages.ts` | `handleMessage*()` |
| **도구 핸들러** | `src/agents/pi-embedded-subscribe.handlers.tools.ts` | `handleToolExecution*()` |
| **세션 컴팩션** | `src/agents/pi-embedded-runner/compact.ts` | `compactEmbeddedPiSessionDirect()` |
| **히스토리 관리** | `src/agents/pi-embedded-runner/history.ts` | `limitHistoryTurns()` |
| **응답 페이로드** | `src/agents/pi-embedded-runner/run/payloads.ts` | `buildEmbeddedRunPayloads()` |
| **이미지 처리** | `src/agents/pi-embedded-runner/run/images.ts` | `detectAndLoadPromptImages()` |
| **응답 전달** | `src/commands/agent/delivery.ts` | `deliverAgentCommandResult()` |
| **아웃바운드 전달** | `src/infra/outbound/deliver.ts` | `deliverOutboundPayloads()` |
| **텔레그램 디스패치** | `src/telegram/bot-message-dispatch.ts` | `dispatchTelegramMessage()` |
| **게이트웨이 채팅** | `src/gateway/server-chat.ts` | `createChatRunRegistry()` |

---

## 요약: 쿼리 한 건의 여정

```
사용자 "파일 목록 보여줘" 입력
  ↓
[1] CLI/Channel → Gateway HTTP 서버 도착
  ↓
[2] RPC 라우팅 → agent 핸들러 → 검증/중복제거/세션해석
  ↓
[3] 즉시 "accepted" 응답 → 백그라운드 에이전트 실행 시작
  ↓
[4] 세션 히스토리 로드 → 시스템 프롬프트 구성 → 도구 등록
  ↓
[5] LLM API 호출 (시스템프롬프트 + 대화기록 + 도구정의 + 사용자메시지)
  ↓
[6] LLM 응답: "파일 목록을 확인하겠습니다" + tool_use(exec: "ls -la")
  ↓
[7] exec 도구 실행 → "total 48\ndrwxr-xr-x  5 user ..."
  ↓
[8] tool_result를 대화에 추가 → LLM 재호출
  ↓
[9] LLM 최종 응답: "현재 디렉토리의 파일 목록입니다:\n..."
  ↓
[10] 응답 페이로드 조합 → 채널별 포맷 변환 → 전송
  ↓
사용자에게 최종 응답 도달
```
