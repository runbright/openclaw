---
summary: "전송, Gateway 및 에이전트 응답을 위한 이미지 및 미디어 처리 규칙"
read_when:
  - 미디어 파이프라인 또는 첨부파일 수정 시
title: "이미지 및 미디어 지원"
---

# 이미지 및 미디어 지원 -- 2025-12-05

WhatsApp 채널은 **Baileys Web**을 통해 실행됩니다. 이 문서는 전송, Gateway 및 에이전트 응답을 위한 현재 미디어 처리 규칙을 기록합니다.

## 목표

- `openclaw message send --media`를 통해 선택적 캡션과 함께 미디어 전송.
- 웹 인박스의 자동 응답이 텍스트와 함께 미디어를 포함할 수 있도록 허용.
- 타입별 제한을 합리적이고 예측 가능하게 유지.

## CLI 인터페이스

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media`는 선택사항; 미디어 전용 전송을 위해 캡션이 비어있을 수 있음.
  - `--dry-run`은 해석된 페이로드를 출력; `--json`은 `{ channel, to, messageId, mediaUrl, caption }`을 생성.

## WhatsApp Web 채널 동작

- 입력: 로컬 파일 경로 **또는** HTTP(S) URL.
- 흐름: Buffer로 로드, 미디어 종류 감지, 올바른 페이로드 구성:
  - **이미지:** JPEG로 리사이즈 및 재압축 (최대 측면 2048px) `agents.defaults.mediaMaxMb` (기본값 5MB) 대상, 6MB 상한.
  - **오디오/음성/비디오:** 16MB까지 패스스루; 오디오는 음성 메모로 전송 (`ptt: true`).
  - **문서:** 나머지 모든 것, 100MB까지, 가능한 경우 파일명 보존.
- WhatsApp GIF 스타일 재생: `gifPlayback: true`로 MP4 전송 (CLI: `--gif-playback`) 모바일 클라이언트에서 인라인 루프.
- MIME 감지는 매직 바이트, 헤더, 파일 확장자 순으로 선호.
- 캡션은 `--message` 또는 `reply.text`에서 가져옴; 빈 캡션 허용.
- 로깅: 비상세 모드에서 결과 표시; 상세 모드에서 크기와 소스 경로/URL 포함.

## 자동 응답 파이프라인

- `getReplyFromConfig`는 `{ text?, mediaUrl?, mediaUrls? }`를 반환합니다.
- 미디어가 있으면 웹 전송자가 `openclaw message send`와 동일한 파이프라인을 사용하여 로컬 경로 또는 URL을 해석합니다.
- 제공된 경우 여러 미디어 항목이 순차적으로 전송됩니다.

## 수신 미디어에서 명령으로 (Pi)

- 수신 웹 메시지에 미디어가 포함되면, OpenClaw는 임시 파일로 다운로드하고 템플릿 변수를 노출합니다:
  - `{{MediaUrl}}` 수신 미디어의 유사 URL.
  - `{{MediaPath}}` 명령 실행 전에 작성된 로컬 임시 경로.
- 세션별 Docker 샌드박스가 활성화되면, 수신 미디어가 샌드박스 워크스페이스에 복사되고 `MediaPath`/`MediaUrl`이 `media/inbound/<filename>` 같은 상대 경로로 다시 작성됩니다.
- 미디어 이해 (`tools.media.*` 또는 공유 `tools.media.models`를 통해 구성된 경우)는 템플릿 전에 실행되며 `Body`에 `[Image]`, `[Audio]`, `[Video]` 블록을 삽입할 수 있습니다.
  - 오디오는 `{{Transcript}}`를 설정하고 명령 파싱에 트랜스크립트를 사용하여 슬래시 명령이 계속 작동합니다.
  - 비디오 및 이미지 설명은 명령 파싱을 위해 캡션 텍스트를 보존합니다.
- 기본적으로 첫 번째 일치하는 이미지/오디오/비디오 첨부파일만 처리됩니다; 여러 첨부파일을 처리하려면 `tools.media.<cap>.attachments`를 설정하세요.

## 제한 및 오류

**아웃바운드 전송 상한 (WhatsApp 웹 전송)**

- 이미지: 재압축 후 ~6MB 상한.
- 오디오/음성/비디오: 16MB 상한; 문서: 100MB 상한.
- 초과 또는 읽을 수 없는 미디어 -> 로그에 명확한 오류, 응답 건너뜀.

**미디어 이해 상한 (트랜스크립션/설명)**

- 이미지 기본값: 10MB (`tools.media.image.maxBytes`).
- 오디오 기본값: 20MB (`tools.media.audio.maxBytes`).
- 비디오 기본값: 50MB (`tools.media.video.maxBytes`).
- 초과 미디어는 이해를 건너뛰지만, 원본 본문으로 응답은 계속됩니다.

## 테스트 참고사항

- 이미지/오디오/문서 케이스에 대한 전송 + 응답 흐름 커버.
- 이미지 재압축 (크기 제한) 및 오디오 음성 메모 플래그 유효성 검사.
- 다중 미디어 응답이 순차 전송으로 전개되는지 확인.
