---
summary: "`openclaw dns` CLI 참조 (광역 디스커버리 도우미)"
read_when:
  - Tailscale + CoreDNS를 통한 광역 디스커버리(DNS-SD)가 필요한 경우
  - 커스텀 디스커버리 도메인(예: openclaw.internal)에 대한 분할 DNS를 설정하는 경우
title: "dns"
---

# `openclaw dns`

광역 디스커버리를 위한 DNS 도우미 (Tailscale + CoreDNS). 현재 macOS + Homebrew CoreDNS에 초점을 맞추고 있습니다.

관련 항목:

- Gateway 디스커버리: [Discovery](/gateway/discovery)
- 광역 디스커버리 설정: [Configuration](/gateway/configuration)

## 설정

```bash
openclaw dns setup
openclaw dns setup --apply
```
