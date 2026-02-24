---
summary: "채널별 위치 정보(Telegram, WhatsApp, Matrix) 파싱 방식 및 상세 컨텍스트 필드 안내"
read_when:
  - 채널의 위치 정보 파싱 로직을 수정할 때
  - 에이전트 프롬프트나 도구에서 위치 정보를 활용하고 싶을 때
title: "위치 데이터 파싱"
---

# 채널 위치 데이터 파싱 (Channel Location Parsing)

OpenClaw는 채팅 채널에서 공유된 위치 정보를 정규화하여 다음과 같이 처리합니다:

- 에이전트가 읽을 수 있도록 사람이 읽기 쉬운 텍스트 형식으로 메시지 본문에 추가합니다.
- 자동 응답 컨텍스트 페이로드의 정형화된 필드(Structured fields)에 데이터를 담아 전달합니다.

현재 지원되는 채널:
- **Telegram** (위치 핀, 장소(Venue), 실시간 위치)
- **WhatsApp** (위치 메시지, 실시간 위치 메시지)
- **Matrix** (`geo_uri` 형식의 위치 정보)

## 텍스트 형식 (Text Formatting)

위치 정보는 대화 맥락에 다음과 같은 친숙한 형식으로 표시됩니다:

- **위치 핀 (Pin)**:
  - `📍 48.858844, 2.294351 ±12m`
- **이름이 지정된 장소 (Named place)**:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- **실시간 위치 (Live share)**:
  - `🛰 실시간 위치: 48.858844, 2.294351 ±12m`

발신자가 멘트(자막)를 함께 보낸 경우 다음 줄에 추가됩니다.

## 컨텍스트 필드 (Context Fields)

위치 정보가 포함된 경우 에이전트는 다음 필드들을 활용할 수 있습니다:

- `LocationLat`: 위도 (숫자)
- `LocationLon`: 경도 (숫자)
- `LocationAccuracy`: 정확도 (미터 단위, 숫자)
- `LocationName`: 장소 이름
- `LocationAddress`: 상세 주소
- `LocationSource`: 데이터 출처 (`pin | place | live`)
- `LocationIsLive`: 실시간 위치 여부 (불리언)

## 채널별 특이사항

- **Telegram**: 장소 정보는 `LocationName`과 `LocationAddress`에 매핑됩니다.
- **WhatsApp**: 위치 메시지의 코멘트(Comment)나 실시간 위치의 캡션(Caption)이 텍스트 본문에 추가됩니다.
- **Matrix**: Altitude(고도) 정보는 무시되며, 항상 실시간 위치가 아닌 핀 위치로 처리됩니다.
---
