# DNS Providers

This project supports two DNS backends in the optional `dns` role:
- `cloudflare`
- `regru`

## Recommendation

If you can choose freely, prefer `cloudflare` as the default backend.

Use `regru` when:
- your authoritative DNS is already hosted at Reg.ru
- you do not want to move the zone elsewhere
- the narrower record automation is sufficient for your setup

## Comparison

| Provider | Status In This Project | Strengths | Tradeoffs | Recommended Default |
| --- | --- | --- | --- | --- |
| `cloudflare` | Fully implemented through `community.general.cloudflare_dns` | Clean Ansible integration, simpler idempotent behavior, easier long-term maintenance | Requires `community.general` on the control host, separate provider account/token management | Yes |
| `regru` | Implemented through direct `REG.API 2` calls with `uri` | Useful when the zone already lives at Reg.ru, no DNS migration required | Narrower implementation, less elegant than native collection modules, more API-shape coupling | Only if your DNS is already there |

## Practical Notes

### Cloudflare

- Best fit if you want the most maintainable Ansible path.
- Supports the stack well for:
  - apex `A`
  - upload host `A`
  - conference host `A`
  - optional SRV records
- Keep `proxied: false` for XMPP-related records unless you have a very specific reason to do otherwise.

### Reg.ru

- Best fit when your DNS is already managed at Reg.ru and you want to avoid moving the zone.
- Current backend is intentionally focused on the records this stack needs:
  - `A`
  - `AAAA`
  - `CNAME`
  - basic `SRV`
- It is a practical automation layer, not a full generic DNS provider abstraction.

## Inventory Examples

- Cloudflare:
  - `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.main.yml.example`
  - `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.vault.yml.example`
- Reg.ru:
  - `inventories/production/group_vars/xmpp_hosts/examples/regru.main.yml.example`
  - `inventories/production/group_vars/xmpp_hosts/examples/regru.vault.yml.example`
