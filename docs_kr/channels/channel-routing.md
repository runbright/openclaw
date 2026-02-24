---
summary: "채널별 라우팅 규칙(WhatsApp, Telegram, Discord, Slack) 및 세션 관리 가이드"
read_when:
  - 채널로부터 들어온 메시지가 어떤 에이전트(두뇌)에게 전달되는지 알고 싶을 때
  - 여러 에이전트를 동시에 운영하며 대화 상대를 분리하고 싶을 때
title: "채널 라우팅"
---

# 채널 및 라우팅 (Channels & Routing)

OpenClaw는 메시지가 들어온 채널로 다시 답변을 보내는 **결정적 라우팅** 방식을 사용합니다. AI 모델이 직접 채널을 선택하는 것이 아니라, 설정된 규칙에 따라 자동으로 경로가 정해집니다.

## 주요 개념
- **채널(Channel)**: 메신저 종류 (WhatsApp, Telegram 등)
- **에이전트 ID (AgentId)**: 독립된 작업 공간과 기억(세션)을 가진 AI '두뇌'
- **세션 키 (SessionKey)**: 대화 맥락이 저장되는 저장소의 고유 이름

## 세션 키의 구조
- **개인 메시지(DM)**: 대부분 메인 세션(`agent:main:main`)으로 통합됩니다.
- **그룹 대화**: 채널별, 그룹별로 독립된 세션을 가집니다. (`agent:main:telegram:group:ID`)
- **스레드(Thread)**: 슬랙이나 디스코드의 스레드는 해당 대화의 ID가 뒤에 추가되어 별도의 맥락을 유지합니다.

## 에이전트 선택 규칙 (우선순위)
메시지가 들어오면 OpenClaw는 아래 순서대로 어떤 에이전트가 답변할지 결정합니다:
1.  **정확한 피어(Peer) 매칭**: 특정 그룹이나 특정인(`id`)에게 지정된 에이전트
2.  **디스코드 역할(Roles) 매칭**: 특정 권한을 가진 사용자 전용 에이전트
3.  **팀/계정(Team/Account) 매칭**: 특정 슬랙 워크스페이스나 텔레그램 계정 전용 에이전트
4.  **채널 매칭**: 텔레그램 전체 전용 에이전트 등
5.  **기본 에이전트**: 위 조건에 해당하지 않을 때 답변하는 기본 에이전트 (보통 `main`)

## 바인딩(Bindings) 설정 예시
`openclaw.json`에서 특정 채널이나 대상을 특정 에이전트에게 연결할 수 있습니다.
```json5
{
  "agents": [
    { "id": "support", "name": "고객지원팀", "workspace": "./support-docs" }
  ],
  "bindings": [
    { "match": { "channel": "slack", "teamId": "T123" }, "agentId": "support" },
    { "match": { "channel": "telegram", "peer": { "kind": "group", "id": "-100123" } }, "agentId": "support" }
  ]
}
```

## 세션 저장소
대화 기록은 기본적으로 `~/.openclaw/agents/<에이전트ID>/sessions/` 경로에 JSON 파일로 안전하게 저장됩니다.

---
더 복잡한 구성은 [멀티 에이전트 가이드](/concepts/multi-agent)를 참고하세요.
---
