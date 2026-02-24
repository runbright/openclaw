---
summary: "게이트웨이 서비스 실행, 수명 주기 및 운영 관리 가이드"
read_when:
  - 게이트웨이 프로세스를 실행하거나 디버깅할 때
title: "게이트웨이 운영 가이드"
---

# 게이트웨이 운영 가이드 (Gateway Runbook)

OpenClaw 게이트웨이 서비스의 시작부터 운영 관리까지 아우르는 실무 가이드입니다.

## 빠른 시작 (로컬 환경)

1. **게이트웨이 실행**:
   ```bash
   openclaw gateway --port 18789
   ```
2. **상태 확인**:
   ```bash
   openclaw status      # 시스템 요약 정보
   openclaw logs --follow  # 실시간 로그 확인
   ```
3. **채널 진단**:
   ```bash
   openclaw channels status --probe
   ```

## 핵심 동작 방식

- **통합 포트**: WebSocket(제어/RPC), HTTP API(OpenAI 호환), 관리 UI, 웹훅 기능이 모두 하나의 포트(기본 `18789`)에서 작동합니다.
- **인증 필수**: 보안을 위해 토큰이나 비밀번호 기반의 인증이 기본적으로 요구됩니다.
- **실시간 적용 (Hot Reload)**: 설정 파일을 수정하면 게이트웨이가 감지하여 즉시 반영합니다. 핵심 설정 변경 시에는 자동으로 재시작됩니다.

## 런타임 제어 명령어

```bash
openclaw gateway status   # 실행 상태 확인
openclaw gateway install  # 시스템 서비스로 등록 (자동 실행)
openclaw gateway restart  # 게이트웨이 재시작
openclaw gateway stop     # 게이트웨이 중지
openclaw doctor           # 종합 진단 및 자동 복구
```

## 원격 접속 (Remote Access)

가장 권장되는 방식은 **Tailscale/VPN**을 통한 접속입니다. 보안상 직접 포트를 열기보다는 **SSH 터널링**을 활용하는 것이 좋습니다.

```bash
# SSH 터널링 예시
ssh -N -L 18789:127.0.0.1:18789 사용자@호스트
```
터널링을 통하더라도 게이트웨이에 설정된 인증 토큰은 동일하게 필요합니다.

## 주요 문제 해결 (에러 시그니처)

| 증상 / 에러 메시지 | 원인 및 해결 방법 |
| :--- | :--- |
| `EADDRINUSE` / 이미 실행 중 | 다른 게이트웨이가 이미 해당 포트를 사용 중입니다. |
| `refusing to bind ... without auth` | 루프백이 아닌 외부 접속을 허용하려면 인증 토큰이 설정되어야 합니다. |
| `unauthorized` | 클라이언트와 게이트웨이 간의 인증 정보가 일치하지 않습니다. |

더 자세한 진단 방법은 [트러블슈팅 가이드](/gateway/troubleshooting)를 참고하십시오.

---
**관련 문서**:
- [설정 가이드](/gateway/configuration)
- [상태 점검](/gateway/health)
- [닥터 (진단 도구)](/gateway/doctor)
- [배경 프로세스 관리](/gateway/background-process)
---
