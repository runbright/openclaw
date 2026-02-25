---
summary: "음성 메시지 인식을 위한 딥그램(Deepgram) 설정 가이드"
read_when:
  - 수신된 오디오 메시지를 텍스트로 변환하고 싶을 때
  - Deepgram STT(Speech-to-Text) 설정을 확인해야 할 때
title: "딥그램 (Deepgram)"
---

# 딥그램 (Deepgram / Audio Transcription)

딥그램(Deepgram)은 강력한 음성 인식(STT) API입니다. OpenClaw에서는 수신된 **오디오 파일이나 음성 메시지를 텍스트로 변환**하여 에이전트에게 전달하는 용도로 사용됩니다.

## 작동 방식
사용자가 음성 메시지를 보내면 OpenClaw가 이를 Deepgram에 업로드하고, 변환된 텍스트(Transcript)를 답변 파이프라인에 주입합니다. 에이전트는 사용자가 말한 내용을 텍스트로 읽고 적절한 답변을 구성할 수 있게 됩니다.

## 빠른 설정 방법

1.  **API 키 설정**: 환경 변수에 Deepgram API 키를 추가합니다.
    ```bash
    DEEPGRAM_API_KEY=dg_...
    ```
2.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "tools": {
        "media": {
          "audio": {
            "enabled": true,
            "models": [{ "provider": "deepgram", "model": "nova-3" }]
          }
        }
      }
    }
    ```

## 주요 옵션
- `model`: 사용할 모델 ID (기본값: `nova-3`)
- `language`: 인식할 언어 힌트 (예: `ko` 또는 `en`)
- `smart_format`: 문장 부호 및 숫자 표기 가독성 개선 여부
- `detect_language`: 언어 자동 감지 기능 활성화 여부

## 참고 사항
- **이미지 및 텍스트**: Deepgram은 오디오 전용입니다. 텍스트 모델 설정은 별도로 필요합니다.
- **인증**: `DEEPGRAM_API_KEY` 환경 변수를 가장 먼저 확인합니다.
- **제한**: 파일 크기 및 하이버네이션 주기는 OpenClaw의 미디어 처리 규칙을 따릅니다.

---
구체적인 API 가이드는 [딥그램 개발자 센터](https://developers.deepgram.com)를 참고하세요.
---
