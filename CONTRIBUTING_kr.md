# OpenClaw 기여 가이드

OpenClaw 프로젝트에 오신 것을 환영합니다! 🦞

## 주요 링크
- **GitHub:** https://github.com/openclaw/openclaw
- **비전:** [`VISION.md`](VISION.md)
- **Discord:** https://discord.gg/qkhbAGHRBT

## 기여 방법
1.  **버그 및 사소한 수정**: 바로 PR(Pull Request)을 올려주세요!
2.  **새로운 기능 및 아키텍처**: [GitHub Discussion](https://github.com/openclaw/openclaw/discussions)을 생성하거나 Discord에서 먼저 논의해 주세요.
3.  **질문**: Discord의 [#help](https://discord.com/channels/1456350064065904867/1459642797895319552) 채널을 이용해 주세요.

## PR 제출 전 확인 사항
- 로컬 OpenClaw 인스턴스에서 직접 테스트를 완료했는지 확인하세요.
- 테스트 실행: `pnpm build && pnpm check && pnpm test`
- CI 체크를 모두 통과해야 합니다.
- 하나의 PR에는 하나의 주제만 담아주세요. (여러 문제를 섞지 마세요)
- 변경 내용과 이유를 상세히 설명해 주세요.

## AI 도움을 받은 PR 환영! 🤖
Codex, Claude 또는 기타 AI 도구를 사용해 작성하셨나요? **좋습니다! 다만 표시만 해주세요.**

PR에 다음 내용을 포함해 주세요:
- [ ] 제목이나 설명에 'AI 도움' 여부 표시
- [ ] 테스트 수준 명시 (미테스트 / 가벼운 테스트 / 완전 테스트)
- [ ] 가능하면 프롬프트나 세션 로그 포함 (큰 도움이 됩니다!)
- [ ] 본인이 코드가 무엇을 하는지 이해하고 있음을 확인

우리는 AI로 작성된 PR을 환영합니다. 리뷰어가 무엇을 중점적으로 봐야 할지 알 수 있도록 투명하게 공개해 주시면 됩니다.

## 현재 집중 분야 및 로드맵 🗺
우리는 현재 다음 항목들을 우선시하고 있습니다:
- **안정성**: 채널 연결(WhatsApp/Telegram)의 예외 상황 수정.
- **사용자 경험(UX)**: 온보딩 마법사 및 오류 메시지 개선.
- **스킬(Skills)**: 스킬 기여는 OpenClaw 스킬 커뮤니티인 [ClawHub](https://clawhub.ai/)를 방문해 주세요.
- **성능**: 토큰 사용량 최적화 및 압축 로직 개선.

## 취약점 보고
보안 문제를 발견하셨다면 **security@openclaw.ai**로 직접 이메일을 보내주시거나 관련 저장소의 이슈로 보고해 주세요. 보고 시에는 재현 방법(Reproduction steps)과 해결 권장 사항이 반드시 포함되어야 합니다.

---
기여해 주시는 모든 분께 감사드립니다!
---
