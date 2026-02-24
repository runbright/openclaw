# OpenClaw에 기여하기

랍스터 탱크에 오신 것을 환영합니다!

## 빠른 링크

- **GitHub:** https://github.com/openclaw/openclaw
- **비전:** [`VISION.md`](VISION.md)
- **Discord:** https://discord.gg/qkhbAGHRBT
- **X/Twitter:** [@steipete](https://x.com/steipete) / [@openclaw](https://x.com/openclaw)

## 관리자

- **Peter Steinberger** - 자비로운 독재자
  - GitHub: [@steipete](https://github.com/steipete) · X: [@steipete](https://x.com/steipete)

- **Shadow** - Discord 서브시스템, Discord 관리자, Clawhub, 모든 커뮤니티 모더레이션
  - GitHub: [@thewilloftheshadow](https://github.com/thewilloftheshadow) · X: [@4shad0wed](https://x.com/4shad0wed)

- **Vignesh** - Memory (QMD), 포멀 모델링, TUI, IRC, Lobster
  - GitHub: [@vignesh07](https://github.com/vignesh07) · X: [@\_vgnsh](https://x.com/_vgnsh)

- **Jos** - Telegram, API, Nix 모드
  - GitHub: [@joshp123](https://github.com/joshp123) · X: [@jjpcodes](https://x.com/jjpcodes)

- **Ayaan Zaidi** - Telegram 서브시스템, iOS 앱
  - GitHub: [@obviyus](https://github.com/obviyus) · X: [@0bviyus](https://x.com/0bviyus)

- **Tyler Yust** - 에이전트/서브에이전트, cron, BlueBubbles, macOS 앱
  - GitHub: [@tyler6204](https://github.com/tyler6204) · X: [@tyleryust](https://x.com/tyleryust)

- **Mariano Belinky** - iOS 앱, 보안
  - GitHub: [@mbelinky](https://github.com/mbelinky) · X: [@belimad](https://x.com/belimad)

- **Vincent Koc** - 에이전트, 텔레메트리, 훅, 보안
  - GitHub: [@vincentkoc](https://github.com/vincentkoc) · X: [@vincent_koc](https://x.com/vincent_koc)

- **Seb Slight** - 문서, 에이전트 신뢰성, 런타임 강화
  - GitHub: [@sebslight](https://github.com/sebslight) · X: [@sebslig](https://x.com/sebslig)

- **Christoph Nakazawa** - JS 인프라
  - GitHub: [@cpojer](https://github.com/cpojer) · X: [@cnakazawa](https://x.com/cnakazawa)

- **Gustavo Madeira Santana** - 멀티 에이전트, CLI, 웹 UI
  - GitHub: [@gumadeiras](https://github.com/gumadeiras) · X: [@gumadeiras](https://x.com/gumadeiras)

- **Onur Solmaz** - 에이전트, 개발 워크플로우, ACP 통합, MS Teams
  - GitHub: [@onutc](https://github.com/onutc), [@osolmaz](https://github.com/osolmaz) · X: [@onusoz](https://x.com/onusoz)

## 기여하는 방법

1. **버그 및 작은 수정** → PR을 여세요!
2. **새 기능 / 아키텍처** → [GitHub Discussion](https://github.com/openclaw/openclaw/discussions)을 시작하거나 Discord에서 먼저 문의하세요
3. **질문** → Discord [#help](https://discord.com/channels/1456350064065904867/1459642797895319552) / [#users-helping-users](https://discord.com/channels/1456350064065904867/1459007081603403828)

## PR 전 체크리스트

- OpenClaw 인스턴스로 로컬 테스트
- 테스트 실행: `pnpm build && pnpm check && pnpm test`
- CI 검사 통과 확인
- PR은 집중적으로 유지 (PR당 하나의 작업; 관련 없는 것을 혼합하지 않기)
- 무엇을 했고 왜 했는지 설명

## Control UI 데코레이터

Control UI는 **레거시** 데코레이터와 함께 Lit를 사용합니다 (현재 Rollup 파싱이 표준 데코레이터에 필요한
`accessor` 필드를 지원하지 않음). 반응형 필드를 추가할 때는 레거시 스타일을 유지하세요:

```ts
@state() foo = "bar";
@property({ type: Number }) count = 0;
```

루트 `tsconfig.json`은 레거시 데코레이터(`experimentalDecorators: true`)와
`useDefineForClassFields: false`로 설정되어 있습니다. UI 빌드 도구를 표준 데코레이터를 지원하도록
업데이트하지 않는 한 이 설정을 변경하지 마세요.

## AI/바이브 코딩 PR 환영합니다!

Codex, Claude, 또는 기타 AI 도구로 빌드하셨나요? **좋습니다 - 표시만 해주세요!**

PR에 다음을 포함해 주세요:

- [ ] PR 제목 또는 설명에 AI 지원 표시
- [ ] 테스트 수준 표기 (미테스트 / 간단히 테스트 / 완전히 테스트)
- [ ] 가능하면 프롬프트 또는 세션 로그 포함 (매우 유용합니다!)
- [ ] 코드가 무엇을 하는지 이해하고 있음을 확인

AI PR은 여기서 일급 시민입니다. 리뷰어가 무엇을 살펴봐야 하는지 알 수 있도록 투명성만 원합니다.

## 현재 집중 분야 및 로드맵

현재 우선순위:

- **안정성**: 채널 연결(WhatsApp/Telegram)의 엣지 케이스 수정.
- **UX**: 온보딩 마법사와 에러 메시지 개선.
- **Skills**: 스킬 기여는 [ClawHub](https://clawhub.ai/)로 이동하세요 — OpenClaw 스킬을 위한 커뮤니티 허브입니다.
- **성능**: 토큰 사용량 최적화 및 컴팩션 로직 개선.

"good first issue" 라벨이 있는 [GitHub Issues](https://github.com/openclaw/openclaw/issues)를 확인하세요!

## 관리자

관리자 팀을 선별적으로 확장하고 있습니다.
코드, 문서, 커뮤니티를 통해 OpenClaw의 방향을 형성하는 데 도움을 주고 싶은 경험 있는 기여자라면 연락을 기다리고 있습니다.

관리자가 되는 것은 명예직이 아닌 책임입니다. 이슈 분류, PR 검토, 프로젝트 발전을 위한 적극적이고 일관된 참여를 기대합니다.

관심이 있으시면 contributing@openclaw.ai로 다음을 보내주세요:

- OpenClaw에 대한 PR 링크 (없다면 먼저 그것부터 시작하세요)
- 유지관리하거나 활발히 기여하는 오픈소스 프로젝트 링크
- GitHub, Discord, X/Twitter 핸들
- 간략한 자기소개: 배경, 경험, 관심 분야
- 구사하는 언어와 거주 지역
- 현실적으로 투자할 수 있는 시간

엔지니어링, 문서, 커뮤니티 관리 등 모든 분야의 인재를 환영합니다.
모든 사람이 직접 작성한 지원서를 신중하게 검토하며 관리자를 천천히 신중하게 추가합니다.
응답까지 몇 주가 걸릴 수 있습니다.

## 취약점 신고

보안 신고를 심각하게 받아들입니다. 문제가 있는 저장소에 직접 취약점을 신고하세요:

- **Core CLI 및 게이트웨이** — [openclaw/openclaw](https://github.com/openclaw/openclaw)
- **macOS 데스크톱 앱** — [openclaw/openclaw](https://github.com/openclaw/openclaw) (apps/macos)
- **iOS 앱** — [openclaw/openclaw](https://github.com/openclaw/openclaw) (apps/ios)
- **Android 앱** — [openclaw/openclaw](https://github.com/openclaw/openclaw) (apps/android)
- **ClawHub** — [openclaw/clawhub](https://github.com/openclaw/clawhub)
- **신뢰 및 위협 모델** — [openclaw/trust](https://github.com/openclaw/trust)

특정 저장소에 해당하지 않거나 확실하지 않은 경우 **security@openclaw.ai**로 이메일을 보내주시면 적절한 곳으로 전달하겠습니다.

### 신고 시 필수 항목

1. **제목**
2. **심각도 평가**
3. **영향**
4. **영향받는 구성요소**
5. **기술적 재현 방법**
6. **입증된 영향**
7. **환경**
8. **수정 제안**

재현 단계, 입증된 영향, 수정 제안이 없는 신고는 우선순위가 낮아집니다. AI 생성 스캐너 결과의 양이 많기 때문에 문제를 이해하는 연구자의 검증된 신고를 받아야 합니다.
