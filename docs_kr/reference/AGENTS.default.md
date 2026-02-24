---
title: "기본 AGENTS.md"
summary: "개인 비서 설정을 위한 기본 OpenClaw 에이전트 지침 및 스킬 목록"
read_when:
  - Starting a new OpenClaw agent session
  - Enabling or auditing default skills
---

# AGENTS.md — OpenClaw 개인 비서 (기본)

## 첫 실행 (권장)

OpenClaw은 에이전트를 위한 전용 워크스페이스 디렉토리를 사용합니다. 기본값: `~/.openclaw/workspace` (`agents.defaults.workspace`로 설정 가능).

1. 워크스페이스 생성 (아직 없는 경우):

```bash
mkdir -p ~/.openclaw/workspace
```

2. 기본 워크스페이스 템플릿을 워크스페이스에 복사:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. 선택 사항: 개인 비서 스킬 목록을 사용하려면 AGENTS.md를 이 파일로 교체:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. 선택 사항: `agents.defaults.workspace`를 설정하여 다른 워크스페이스 선택 (`~` 지원):

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## 안전 기본값

- 디렉토리나 비밀정보를 채팅에 출력하지 마세요.
- 명시적으로 요청받지 않는 한 파괴적인 명령을 실행하지 마세요.
- 외부 메시징 표면에 부분적/스트리밍 응답을 보내지 마세요 (최종 응답만 전송).

## 세션 시작 (필수)

- `SOUL.md`, `USER.md`, `memory.md`, 그리고 `memory/`의 오늘+어제 파일을 읽으세요.
- 응답하기 전에 수행하세요.

## 소울 (필수)

- `SOUL.md`는 정체성, 톤, 경계를 정의합니다. 최신 상태를 유지하세요.
- `SOUL.md`를 변경하면 사용자에게 알리세요.
- 매 세션마다 새로운 인스턴스입니다; 연속성은 이 파일들에 있습니다.

## 공유 공간 (권장)

- 사용자의 목소리가 아닙니다; 그룹 채팅이나 공개 채널에서 주의하세요.
- 개인 데이터, 연락처 정보, 내부 메모를 공유하지 마세요.

## 메모리 시스템 (권장)

- 일일 로그: `memory/YYYY-MM-DD.md` (필요시 `memory/` 생성).
- 장기 메모리: `memory.md`에 지속적인 사실, 선호도, 결정을 저장.
- 세션 시작 시 오늘 + 어제 + `memory.md`가 있으면 읽기.
- 캡처 항목: 결정, 선호도, 제약 조건, 미결 사항.
- 명시적으로 요청받지 않는 한 비밀정보는 피하세요.

## 도구 및 스킬

- 도구는 스킬에 있습니다; 필요할 때 각 스킬의 `SKILL.md`를 따르세요.
- 환경별 메모는 `TOOLS.md` (스킬을 위한 메모)에 보관하세요.

## 백업 팁 (권장)

이 워크스페이스를 Clawd의 "메모리"로 취급한다면, git 저장소(가급적 비공개)로 만들어 `AGENTS.md`와 메모리 파일들이 백업되도록 하세요.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# 선택 사항: 비공개 리모트 추가 + push
```

## OpenClaw이 하는 일

- WhatsApp 게이트웨이 + Pi 코딩 에이전트를 실행하여 비서가 채팅을 읽고/쓰고, 컨텍스트를 가져오고, 호스트 Mac을 통해 스킬을 실행할 수 있습니다.
- macOS 앱은 권한(화면 녹화, 알림, 마이크)을 관리하고 번들된 바이너리를 통해 `openclaw` CLI를 노출합니다.
- 다이렉트 채팅은 기본적으로 에이전트의 `main` 세션으로 통합됩니다; 그룹은 `agent:<agentId>:<channel>:group:<id>`로 격리됩니다 (방/채널: `agent:<agentId>:<channel>:channel:<id>`); 하트비트는 백그라운드 작업을 유지합니다.

## 핵심 스킬 (설정 → 스킬에서 활성화)

- **mcporter** — 외부 스킬 백엔드를 관리하기 위한 도구 서버 런타임/CLI.
- **Peekaboo** — 선택적 AI 비전 분석이 포함된 빠른 macOS 스크린샷.
- **camsnap** — RTSP/ONVIF 보안 카메라에서 프레임, 클립 또는 모션 알림을 캡처.
- **oracle** — 세션 리플레이 및 브라우저 제어가 포함된 OpenAI 호환 에이전트 CLI.
- **eightctl** — 터미널에서 수면을 제어.
- **imsg** — iMessage 및 SMS 전송, 읽기, 스트리밍.
- **wacli** — WhatsApp CLI: 동기화, 검색, 전송.
- **discord** — Discord 액션: 반응, 스티커, 투표. `user:<id>` 또는 `channel:<id>` 대상 사용 (숫자만 있는 id는 모호함).
- **gog** — Google Suite CLI: Gmail, Calendar, Drive, Contacts.
- **spotify-player** — 검색/큐/재생 제어를 위한 터미널 Spotify 클라이언트.
- **sag** — mac 스타일 say UX를 갖춘 ElevenLabs 음성; 기본적으로 스피커로 스트리밍.
- **Sonos CLI** — 스크립트에서 Sonos 스피커 제어 (검색/상태/재생/볼륨/그룹핑).
- **blucli** — 스크립트에서 BluOS 플레이어 재생, 그룹, 자동화.
- **OpenHue CLI** — 씬 및 자동화를 위한 Philips Hue 조명 제어.
- **OpenAI Whisper** — 빠른 받아쓰기 및 음성 메일 전사를 위한 로컬 음성-텍스트 변환.
- **Gemini CLI** — 터미널에서 빠른 Q&A를 위한 Google Gemini 모델.
- **agent-tools** — 자동화 및 헬퍼 스크립트를 위한 유틸리티 툴킷.

## 사용 참고사항

- 스크립팅에는 `openclaw` CLI를 선호하세요; mac 앱이 권한을 처리합니다.
- 스킬 탭에서 설치를 실행하세요; 바이너리가 이미 있으면 버튼이 숨겨집니다.
- 비서가 리마인더를 예약하고, 받은 편지함을 모니터링하고, 카메라 캡처를 트리거할 수 있도록 하트비트를 활성화 상태로 유지하세요.
- Canvas UI는 네이티브 오버레이와 함께 전체 화면으로 실행됩니다. 중요한 컨트롤을 왼쪽 상단/오른쪽 상단/하단 가장자리에 배치하지 마세요; 레이아웃에 명시적 여백을 추가하고 safe-area insets에 의존하지 마세요.
- 브라우저 기반 인증에는 OpenClaw이 관리하는 Chrome 프로필과 함께 `openclaw browser` (tabs/status/screenshot)를 사용하세요.
- DOM 검사에는 `openclaw browser eval|query|dom|snapshot` (머신 출력이 필요할 때 `--json`/`--out`)을 사용하세요.
- 상호작용에는 `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run`을 사용하세요 (click/type는 snapshot 참조 필요; CSS 선택자에는 `evaluate` 사용).
