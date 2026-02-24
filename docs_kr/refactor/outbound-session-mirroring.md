---
title: 아웃바운드 세션 미러링 리팩토링 (Issue #1520)
description: 아웃바운드 세션 미러링 리팩토링 노트, 결정, 테스트 및 미해결 항목 추적.
summary: "대상 채널 세션에 아웃바운드 전송을 미러링하기 위한 리팩토링 노트"
read_when:
  - Working on outbound transcript/session mirroring behavior
  - Debugging sessionKey derivation for send/message tool paths
---

# 아웃바운드 세션 미러링 리팩토링 (Issue #1520)

## 상태

- 진행 중.
- 아웃바운드 미러링을 위해 코어 + 플러그인 채널 라우팅 업데이트됨.
- Gateway send가 이제 sessionKey가 생략된 경우 대상 세션을 파생함.

## 컨텍스트

아웃바운드 전송이 대상 채널 세션이 아닌 _현재_ 에이전트 세션(도구 세션 키)에 미러링되었습니다. 인바운드 라우팅은 channel/peer 세션 키를 사용하므로, 아웃바운드 응답이 잘못된 세션에 도착했고 첫 접촉 대상에는 종종 세션 항목이 없었습니다.

## 목표

- 아웃바운드 메시지를 대상 채널 세션 키에 미러링.
- 누락 시 아웃바운드에서 세션 항목 생성.
- 스레드/토픽 범위를 인바운드 세션 키와 정렬 유지.
- 코어 채널과 번들 확장을 포괄.

## 구현 요약

- 새로운 아웃바운드 세션 라우팅 헬퍼:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute`가 `buildAgentSessionKey`를 사용하여 대상 sessionKey 구축 (dmScope + identityLinks).
  - `ensureOutboundSessionEntry`가 `recordSessionMetaFromInbound`를 통해 최소 `MsgContext`를 작성.
- `runMessageAction` (send)가 대상 sessionKey를 파생하고 미러링을 위해 `executeSendAction`에 전달.
- `message-tool`은 더 이상 직접 미러링하지 않음; 현재 세션 키에서 agentId만 해결.
- 플러그인 전송 경로가 파생된 sessionKey를 사용하여 `appendAssistantMessageToSessionTranscript`를 통해 미러링.
- Gateway send가 아무것도 제공되지 않을 때 대상 세션 키를 파생(기본 에이전트)하고 세션 항목을 보장.

## 스레드/토픽 처리

- Slack: replyTo/threadId -> `resolveThreadSessionKeys` (접미사).
- Discord: threadId/replyTo -> `useSuffix=false`인 `resolveThreadSessionKeys`로 인바운드와 일치 (스레드 채널 id가 이미 세션의 범위를 지정).
- Telegram: 토픽 ID가 `buildTelegramGroupPeerId`를 통해 `chatId:topic:<id>`로 매핑.

## 포함된 확장

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- 참고:
  - Mattermost 대상은 이제 DM 세션 키 라우팅을 위해 `@`를 제거.
  - Zalo Personal은 1:1 대상에 DM peer 종류를 사용 (그룹은 `group:`이 있을 때만).
  - BlueBubbles 그룹 대상은 인바운드 세션 키와 일치하도록 `chat_*` 접두사를 제거.
  - Slack 자동 스레드 미러링은 채널 id를 대소문자 구분 없이 일치.
  - Gateway send는 미러링 전에 제공된 세션 키를 소문자로 변환.

## 결정

- **Gateway send 세션 파생**: `sessionKey`가 제공되면 사용. 생략되면 대상 + 기본 에이전트에서 sessionKey를 파생하고 거기에 미러링.
- **세션 항목 생성**: 항상 인바운드 형식에 맞춰 `Provider/From/To/ChatType/AccountId/Originating*`가 정렬된 `recordSessionMetaFromInbound` 사용.
- **대상 정규화**: 아웃바운드 라우팅은 사용 가능한 경우 해결된 대상(`resolveChannelTarget` 이후)을 사용.
- **세션 키 대소문자**: 쓰기 시 및 마이그레이션 중 세션 키를 소문자로 정규화.

## 추가/업데이트된 테스트

- `src/infra/outbound/outbound.test.ts`
  - Slack 스레드 세션 키.
  - Telegram 토픽 세션 키.
  - Discord와 dmScope identityLinks.
- `src/agents/tools/message-tool.test.ts`
  - 세션 키에서 agentId 파생 (sessionKey가 전달되지 않음).
- `src/gateway/server-methods/send.test.ts`
  - 생략 시 세션 키를 파생하고 세션 항목을 생성.

## 미해결 항목 / 후속 작업

- Voice-call 플러그인은 커스텀 `voice:<phone>` 세션 키를 사용. 아웃바운드 매핑은 여기서 표준화되지 않음; message-tool이 voice-call 전송을 지원해야 하는 경우 명시적 매핑 추가.
- 번들 세트를 넘어서 비표준 `From/To` 형식을 사용하는 외부 플러그인이 있는지 확인.

## 변경된 파일

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- 테스트:
  - `src/infra/outbound/outbound.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`
