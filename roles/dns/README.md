# dns

Optional DNS automation role for the XMPP stack.

Current status:
- `cloudflare`: implemented
- `regru`: implemented for standard records and basic SRV support

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_manage_dns` | Enables or disables this role. Default: `false`. |
| `xmpp_stack_dns_provider` | DNS provider backend. Supported now: `cloudflare`, `regru`. |
| `xmpp_stack_dns_zone` | DNS zone to manage. Default: `xmpp_stack_domain`. |
| `xmpp_stack_dns_regru_api_endpoint` | Base URL for the Reg.ru API backend. Default: `https://api.reg.ru/api/regru2`. |
| `xmpp_stack_dns_ttl` | Default TTL for managed records. |
| `xmpp_stack_dns_proxied` | Cloudflare proxy flag for the apex record. Use `false` for XMPP-related records unless you explicitly know otherwise. |
| `xmpp_stack_dns_manage_srv` | Enables SRV record management. Default: `true`. |
| `xmpp_stack_dns_manage_txt` | Enables TXT record management for `_xmppconnect`. Default: `true`. |
| `xmpp_stack_websocket_url` | WebSocket URL published in `_xmppconnect`. Default: `wss://<domain>/ws`. |
| `xmpp_stack_dns_records` | Declarative list of A/AAAA/CNAME-like records to manage. |
| `xmpp_stack_dns_srv_records` | Declarative list of SRV records to manage when enabled. |
| `xmpp_stack_dns_txt_records` | Declarative list of TXT records to manage when enabled. |

### Secrets

| Variable | Description |
| --- | --- |
| `xmpp_stack_dns_secrets.cloudflare_api_token` | Cloudflare API token used by `community.general.cloudflare_dns`. |
| `xmpp_stack_dns_secrets.regru_username` | Reg.ru account username for REG.API 2. |
| `xmpp_stack_dns_secrets.regru_password` | Reg.ru account password for REG.API 2. |

## Notes

- This role is intentionally optional and disabled by default.
- Cloudflare automation uses `community.general.cloudflare_dns`, so the control
  environment needs the `community.general` collection installed.
- The Reg.ru backend talks directly to REG.API 2 via `uri`.
- Reg.ru support is intentionally narrower than the Cloudflare path: it focuses
  on records needed by this stack and basic SRV/TXT handling.
- The default record set is an expanded XMPP profile: apex, `upload`,
  `conference`, `xmpp-client`, optional `xmpp-server`, TURN/STUN SRV, and
  `_xmppconnect` for WebSocket discovery.
- If you use IPv6, set `xmpp_stack_public_ipv6` to generate `AAAA` records.
- If federation or remote MUC access matters, include
  `xmpp_stack_conference_host` in `xmpp_stack_certbot_domains`.
- Shared stack variables are documented in [common](../common/README.md).

## Example Inventory

- Cloudflare main vars:
  [cloudflare.main.yml.example](../../inventories/production/group_vars/xmpp_hosts/examples/cloudflare.main.yml.example)
- Cloudflare vault vars:
  [cloudflare.vault.yml.example](../../inventories/production/group_vars/xmpp_hosts/examples/cloudflare.vault.yml.example)
- Reg.ru main vars:
  [regru.main.yml.example](../../inventories/production/group_vars/xmpp_hosts/examples/regru.main.yml.example)
- Reg.ru vault vars:
  [regru.vault.yml.example](../../inventories/production/group_vars/xmpp_hosts/examples/regru.vault.yml.example)
