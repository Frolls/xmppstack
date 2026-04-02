# coturn

Manages:
- coturn container build context
- local coturn image rebuilds
- TURN Quadlet unit
- conditional service start after certificate bootstrap

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_coturn_image` | Local coturn image tag. |
| `xmpp_stack_build_images` | Whether the role should rebuild the local coturn image. |
| `xmpp_stack_public_ip` | Public IP announced by TURN. Required for working media relay. |
| `xmpp_stack_turn_realm` | TURN realm. Default: XMPP domain. |
| `xmpp_stack_turn_static_auth_secret` | TURN shared secret. Usually sourced from vault. |

## Notes

- This role starts the service only when certificates already exist.
- Image rebuilds are hash-based and only happen when the Containerfile or
  rendered `turnserver.conf` changes.
- TURN listener and relay ports are taken from shared stack variables in
  [common](../common/README.md), so coturn, DNS SRV, and firewall rules can
  stay aligned.
- Shared stack variables such as domain, paths, and certificate storage are
  documented in [common](../common/README.md).
