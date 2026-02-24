---
summary: "OpenClaw 로깅: 롤링 진단 파일 로그 + 통합 로그 프라이버시 플래그"
read_when:
  - macOS 로그 캡처 또는 개인 데이터 로깅 조사
  - 음성 웨이크/세션 수명 주기 문제 디버깅
title: "macOS 로깅"
---

# 로깅 (macOS)

## 롤링 진단 파일 로그 (Debug 패널)

OpenClaw은 macOS 앱 로그를 swift-log를 통해 라우팅하며 (기본적으로 통합 로깅) 지속적인 캡처가 필요할 때 디스크에 로컬 회전 파일 로그를 작성할 수 있습니다.

- 상세도: **Debug 패널 → Logs → App logging → Verbosity**
- 활성화: **Debug 패널 → Logs → App logging → "Write rolling diagnostics log (JSONL)"**
- 위치: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (자동 회전; 이전 파일은 `.1`, `.2`, ... 접미사가 붙음)
- 초기화: **Debug 패널 → Logs → App logging → "Clear"**

참고:

- **기본적으로 꺼져 있습니다**. 활발한 디버깅 중에만 활성화하세요.
- 파일을 민감한 것으로 취급하세요; 검토 없이 공유하지 마세요.

## macOS 통합 로깅 개인 데이터

통합 로깅은 서브시스템이 `privacy -off`를 선택하지 않는 한 대부분의 페이로드를 수정합니다. Peter의 macOS [로깅 프라이버시 이슈](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025) 글에 따르면, 이는 서브시스템 이름으로 키가 지정된 `/Library/Preferences/Logging/Subsystems/`의 plist로 제어됩니다. 새 로그 항목만 플래그를 인식하므로, 문제를 재현하기 전에 활성화하세요.

## OpenClaw (`bot.molt`)에 대해 활성화

- 먼저 plist를 임시 파일에 작성한 다음 root로 원자적으로 설치하세요:

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

- 재부팅이 필요하지 않습니다; logd가 파일을 빠르게 인식하지만, 새 로그 줄만 개인 페이로드를 포함합니다.
- 기존 도우미로 더 풍부한 출력을 확인하세요. 예: `./scripts/clawlog.sh --category WebChat --last 5m`.

## 디버깅 후 비활성화

- 오버라이드 제거: `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`.
- 선택사항으로 `sudo log config --reload`를 실행하여 logd가 즉시 오버라이드를 삭제하도록 강제합니다.
- 이 화면은 전화번호와 메시지 본문을 포함할 수 있음을 기억하세요; 추가 세부 사항이 적극적으로 필요한 동안에만 plist를 유지하세요.
