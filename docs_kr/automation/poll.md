---
summary: "게이트웨이 및 CLI를 통한 설문 조사(Poll) 전송 가이드"
read_when:
  - 채팅 채널(WhatsApp, Discord 등)에 설문 조사를 보내고 싶을 때
  - 설문 조사 관련 명령어 옵션을 확인해야 할 때
title: "설문 조사"
---

# 설문 조사 (Polls)

에이전트나 사용자가 채팅 채널에 투표(설문 조사)를 보낼 수 있는 기능입니다.

## 지원 채널
- **WhatsApp**
- **Discord**
- **Microsoft Teams** (상태 기반 가상 투표)

## CLI 명령어 예시

터미널에서 직접 설문을 보낼 수 있습니다:

```bash
# WhatsApp (기본값)
openclaw message poll --target +821012345678 \
  --poll-question "오늘 점심 메뉴는?" \
  --poll-option "짜장면" --poll-option "짬뽕" --poll-option "볶음밥"

# Discord
openclaw message poll --channel discord --target channel:채널ID \
  --poll-question "진행하시겠습니까?" \
  --poll-option "찬성" --poll-option "반대" \
  --poll-multi # 다중 선택 허용
```

## 주요 옵션 설명
- `--target`: 수신자의 전화번호나 채널 ID.
- `--poll-question`: 설문 질문 내용.
- `--poll-option`: 선택지 (여러 번 사용하여 추가 가능).
- `--poll-multi`: 여러 옵션을 동시에 선택할 수 있게 허용합니다.
- `--poll-duration-hours`: (Discord 전용) 설문 유지 시간 (기본 24시간).

## 에이전트 도구 (Agent Tool)
에이전트가 대화 중에 투표를 생성하게 할 수도 있습니다. `message` 도구를 사용하며, 질문과 옵션들을 제공하면 에이전트가 투표를 채널에 게시합니다.

## 채널별 특징
- **WhatsApp**: 최소 2개에서 최대 12개의 선택지를 지원합니다.
- **Discord**: 최소 2개에서 최대 10개의 선택지를 지원합니다.
- **MS Teams**: 전용 설문 기능이 아닌 '어댑티브 카드(Adaptive Card)'를 통해 시뮬레이션됩니다. (게이트웨이가 켜져 있어야 투표 집계가 가능합니다.)

---
채널별 상세 설정은 [채널 섹션](/channels)을 참고하세요.
---
