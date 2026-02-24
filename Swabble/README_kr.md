# swabble — Speech.framework 웨이크워드 훅 데몬 (macOS 26)

swabble은 Swift 6.2 웨이크워드 훅 데몬입니다. CLI는 macOS 26 (SpeechAnalyzer + SpeechTranscriber)을 대상으로 합니다. 공유 `SwabbleKit` 타겟은 멀티플랫폼이며 iOS/macOS 앱을 위한 웨이크워드 게이팅 유틸리티를 노출합니다.

- **로컬 전용**: Speech.framework 온디바이스 모델; 네트워크 사용 없음.
- **웨이크워드**: 기본 `clawd` (별칭 `claude`), 선택적 `--no-wake` 바이패스.
- **SwabbleKit**: 공유 웨이크 게이트 유틸리티 (음성 세그먼트를 제공할 때 갭 기반 게이팅).
- **훅**: 접두사/환경변수, 쿨다운, min_chars, 타임아웃으로 모든 명령 실행.
- **서비스**: 시작/중지/설치를 위한 launchd 헬퍼 스텁.
- **파일 트랜스크립션**: TXT 또는 SRT (시간 범위 포함, AttributedString 분할 사용).

## 빠른 시작
```bash
# 의존성 설치
brew install swiftformat swiftlint

# 빌드
swift build

# 기본 설정 파일 작성 (~/.config/swabble/config.json)
swift run swabble setup

# 포그라운드 데몬 실행
swift run swabble serve

# 훅 테스트
swift run swabble test-hook "hello world"

# 파일을 SRT로 트랜스크립션
swift run swabble transcribe /path/to/audio.m4a --format srt --output out.srt
```

## 라이브러리로 사용
swabble을 SwiftPM 의존성으로 추가하고 `Swabble` 또는 `SwabbleKit` 프로덕트를 임포트합니다:

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/steipete/swabble.git", branch: "main"),
],
targets: [
    .target(name: "MyApp", dependencies: [
        .product(name: "Swabble", package: "swabble"),     // Speech 파이프라인 (macOS 26+ / iOS 26+)
        .product(name: "SwabbleKit", package: "swabble"),  // 웨이크워드 게이트 유틸리티 (iOS 17+ / macOS 15+)
    ]),
]
```

## CLI
- `serve` — 포그라운드 루프 (마이크 -> 웨이크 -> 훅)
- `transcribe <file>` — 오프라인 트랜스크립션 (txt|srt)
- `test-hook "text"` — 설정된 훅 호출
- `mic list|set <index>` — 입력 장치 나열/선택
- `setup` — 기본 설정 JSON 작성
- `doctor` — Speech 인증 및 장치 가용성 확인
- `health` — `ok` 출력
- `tail-log` — 최근 10개 트랜스크립트
- `status` — 웨이크 상태 + 최근 트랜스크립트 표시
- `service install|uninstall|status` — 사용자 launchd plist (스텁: launchctl 명령 출력)
- `start|stop|restart` — 전체 launchd 연결 전까지 플레이스홀더

모든 명령은 Commander 런타임 플래그(`-v/--verbose`, `--json-output`, `--log-level`)를 허용하며, 해당되는 경우 `--config`도 사용 가능합니다.

## 설정
`~/.config/swabble/config.json` (`setup`으로 자동 생성):
```json
{
  "audio": {"deviceName": "", "deviceIndex": -1, "sampleRate": 16000, "channels": 1},
  "wake": {"enabled": true, "word": "clawd", "aliases": ["claude"]},
  "hook": {
    "command": "",
    "args": [],
    "prefix": "Voice swabble from ${hostname}: ",
    "cooldownSeconds": 1,
    "minCharacters": 24,
    "timeoutSeconds": 5,
    "env": {}
  },
  "logging": {"level": "info", "format": "text"},
  "transcripts": {"enabled": true, "maxEntries": 50},
  "speech": {"localeIdentifier": "en_US", "etiquetteReplacements": false}
}
```

- 설정 경로 재정의: 관련 명령에서 `--config /path/to/config.json`.
- 트랜스크립트는 `~/Library/Application Support/swabble/transcripts.log`에 영속됩니다.

## 훅 프로토콜
웨이크 게이팅된 트랜스크립트가 min_chars 및 쿨다운을 통과하면 swabble이 실행합니다:
```
<command> <args...> "<prefix><text>"
```
환경 변수:
- `SWABBLE_TEXT` — 정리된 트랜스크립트 (웨이크워드 제거됨)
- `SWABBLE_PREFIX` — 렌더링된 접두사 (호스트명 치환됨)
- 추가로 모든 `hook.env` 키/값

## Speech 파이프라인
- `AVAudioEngine` tap -> `BufferConverter` -> `AnalyzerInput` -> `SpeechTranscriber` 모듈이 있는 `SpeechAnalyzer`.
- 임시 및 최종 결과를 요청합니다. CLI는 현재 텍스트 전용 웨이크 게이팅을 사용합니다.
- 첫 시작 시 인가가 요청됩니다. macOS 26 + 새 Speech.framework API가 필요합니다.

## 개발
- 포맷: `./scripts/format.sh` (로컬 `.swiftformat` 사용)
- 린트: `./scripts/lint.sh` (로컬 `.swiftlint.yml` 사용)
- 테스트: `swift test` (swift-testing 패키지 사용)

## 로드맵
- launchd 제어 (load/bootout, PID + 상태 소켓)
- JSON 로깅 + PII 수정 토글
- 더 강력한 웨이크워드 감지 및 제어 소켓 상태/헬스
