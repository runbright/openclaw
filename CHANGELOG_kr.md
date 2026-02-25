# 변경 이력 (Changelog)

## 미출시 (Unreleased)

### 주요 변경 사항 (Breaking)
- 루프백이 아닌 제어 UI는 이제 명시적인 `gateway.controlUi.allowedOrigins` 설정이 필요합니다. 설정이 없으면 보안을 위해 시작이 차단됩니다.

### 수정 사항 (Fixes)
- **온보딩/커스텀 제공업체**: OpenAI 및 Anthropic 호환성 확인 시 토큰 예산(budget)을 늘려 엄격한 기본값으로 인한 오탐을 방지했습니다.
- **WhatsApp/DM 라우팅**: 메인 세션의 마지막 경로 상태 업데이트 로직을 개선하여 격리된 `dmScope` 라우팅을 보호합니다.
- **OpenRouter 제공업체**: '생각(thinking)' 기능이 꺼져 있을 때 `reasoning.effort`를 주입하지 않도록 수정했습니다.
- **보안 관련**: iOS 딥 링크, 음성 호출(Twilio), 세션 HTML 내보내기, 이미지 도구, 샌드박스 설정 등 여러 보안 취약점을 패치하고 강화했습니다. (감사합니다: @GCXWLP, @jiseoung, @allsmog, @tdjackey)

## 2026.2.23 (미출시)

### 주요 변경 사항 (Changes)
- **Kilo Gateway 제공업체 추가**: `kilocode` 제공업체를 정식 지원합니다. (Anthropic Claude Opus 4.6 모델 기본 사용)
- **Vercel AI Gateway**: Claude 모델의 단축어 레이블을 지원합니다.
- **프롬프트 캐싱 문서 추가**: `cacheRetention` 및 모델별 캐시 동작에 대한 상세 가이드를 추가했습니다.
- **세션 관리 강화**: `openclaw sessions cleanup` 명령어를 추가하고 디스크 사용량 제한 기능을 강화했습니다.

---
상세한 전체 변경 이력은 영문 `CHANGELOG.md`를 참조하세요.
---
