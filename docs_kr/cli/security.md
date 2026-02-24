---
summary: "`openclaw security` (일반적인 보안 문제 감사 및 수정) CLI 레퍼런스"
read_when:
  - 구성/상태에 대한 빠른 보안 감사를 실행하고 싶을 때
  - 안전한 "수정" 제안(chmod, 기본값 강화)을 적용하고 싶을 때
title: "security"
---

# `openclaw security`

보안 도구 (감사 + 선택적 수정).

관련 항목:

- 보안 가이드: [보안](/gateway/security)

## 감사

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

감사는 여러 DM 발신자가 기본 세션을 공유할 때 경고하고 **보안 DM 모드**를 권장합니다: 공유 인박스의 경우 `session.dmScope="per-channel-peer"` (또는 다중 계정 채널의 경우 `per-account-channel-peer`).
이는 협력/공유 인박스 강화를 위한 것입니다. 상호 신뢰하지 않는/적대적 운영자가 공유하는 단일 Gateway는 권장 설정이 아닙니다; 별도의 게이트웨이(또는 별도의 OS 사용자/호스트)로 신뢰 경계를 분리하세요.
또한 샌드박싱 없이 웹/브라우저 도구가 활성화된 소형 모델(`<=300B`)이 사용될 때 경고합니다.
웹훅 수신의 경우, `hooks.defaultSessionKey`가 설정되지 않았을 때, 요청 `sessionKey` 재정의가 활성화되었을 때, 그리고 `hooks.allowedSessionKeyPrefixes` 없이 재정의가 활성화되었을 때 경고합니다.
또한 샌드박스 모드가 꺼져 있는데 샌드박스 Docker 설정이 구성되어 있을 때, `gateway.nodes.denyCommands`가 비효과적인 패턴형/알 수 없는 항목을 사용할 때, `gateway.nodes.allowCommands`가 위험한 노드 명령을 명시적으로 활성화할 때, 전역 `tools.profile="minimal"`이 에이전트 도구 프로필에 의해 재정의될 때, 개방 그룹이 샌드박스/워크스페이스 가드 없이 런타임/파일시스템 도구를 노출할 때, 그리고 설치된 확장 플러그인 도구가 허용적인 도구 정책에서 접근 가능할 수 있을 때 경고합니다.
또한 `gateway.allowRealIpFallback=true` (프록시가 잘못 구성된 경우 헤더 스푸핑 위험) 및 `discovery.mdns.mode="full"` (mDNS TXT 레코드를 통한 메타데이터 유출)을 표시합니다.
또한 샌드박스 브라우저가 `sandbox.browser.cdpSourceRange` 없이 Docker `bridge` 네트워크를 사용할 때 경고합니다.
또한 기존 샌드박스 브라우저 Docker 컨테이너에 해시 레이블이 없거나 오래되었을 때 (예: `openclaw.browserConfigEpoch`가 없는 마이그레이션 이전 컨테이너) 경고하고 `openclaw sandbox recreate --browser --all`을 권장합니다.
또한 npm 기반 플러그인/훅 설치 레코드가 고정되지 않았거나, 무결성 메타데이터가 누락되었거나, 현재 설치된 패키지 버전과 차이가 있을 때 경고합니다.
채널 허용 목록이 안정적인 ID 대신 변경 가능한 이름/이메일/태그에 의존할 때 경고합니다 (해당되는 경우 Discord, Slack, Google Chat, MS Teams, Mattermost, IRC 범위).
`gateway.auth.mode="none"`이 공유 비밀 없이 Gateway HTTP API에 접근 가능하게 할 때 경고합니다 (`/tools/invoke` 및 활성화된 모든 `/v1/*` 엔드포인트).
`dangerous`/`dangerously` 접두사가 있는 설정은 명시적 비상 시 운영자 재정의입니다; 이를 활성화하는 것 자체가 보안 취약점 보고는 아닙니다.

## JSON 출력

CI/정책 검사에 `--json`을 사용하세요:

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

`--fix`와 `--json`이 결합되면, 출력에 수정 작업과 최종 보고서가 모두 포함됩니다:

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## `--fix`가 변경하는 내용

`--fix`는 안전하고 결정적인 교정을 적용합니다:

- 일반적인 `groupPolicy="open"`을 `groupPolicy="allowlist"`로 전환 (지원되는 채널의 계정 변형 포함)
- `logging.redactSensitive`를 `"off"`에서 `"tools"`로 설정
- 상태/구성 및 일반 민감 파일(`credentials/*.json`, `auth-profiles.json`, `sessions.json`, 세션 `*.jsonl`)의 권한 강화

`--fix`가 하지 **않는** 것:

- 토큰/비밀번호/API 키 순환
- 도구 비활성화 (`gateway`, `cron`, `exec` 등)
- 게이트웨이 바인드/인증/네트워크 노출 설정 변경
- 플러그인/스킬 제거 또는 재작성
