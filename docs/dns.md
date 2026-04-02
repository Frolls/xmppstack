# DNS

See also:
- [DNS_PROVIDERS.md](./DNS_PROVIDERS.md)
- [DNS_PROVIDERS.ru.md](./DNS_PROVIDERS.ru.md)

Suggested records:
- `A` / `AAAA` for `{{ xmpp_stack_domain }}`
- `A` / `AAAA` for `{{ xmpp_stack_upload_host }}`
- `A` / `AAAA` for `{{ xmpp_stack_conference_host }}`

Expanded SRV records:
- `_xmpp-client._tcp.{{ xmpp_stack_domain }}`
- `_xmpp-server._tcp.{{ xmpp_stack_domain }}` when federation is enabled
- `_turn._udp.{{ xmpp_stack_domain }}`
- `_turn._tcp.{{ xmpp_stack_domain }}`
- `_turns._tcp.{{ xmpp_stack_domain }}`
- `_stun._udp.{{ xmpp_stack_domain }}`
- `_stun._tcp.{{ xmpp_stack_domain }}`
- `_stuns._tcp.{{ xmpp_stack_domain }}`

TXT discovery:
- `_xmppconnect.{{ xmpp_stack_domain }}` with `_xmpp-client-websocket={{ xmpp_stack_websocket_url }}`

If the same host handles all roles, SRV can still improve interoperability.

Automation notes:
- The optional `dns` role can manage these records automatically.
- `cloudflare` is implemented.
- `regru` is implemented through REG.API 2.
- The expanded default profile also manages `_xmppconnect` and TURN/STUN SRV.
- If `xmpp_stack_public_ipv6` is set, `AAAA` records are generated too.
- If `xmpp_stack_enable_nginx: false`, `_xmppconnect` will point to `wss://<domain>:5443/ws`.
