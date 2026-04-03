# common

Shared host preparation for the XMPP stack:
- variable validation
- package installation
- runtime directories
- shared `.env`
- Podman network Quadlet
- operational helper scripts

## Variables

### Required Inventory Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_domain` | Main XMPP domain. |
| `xmpp_stack_upload_host` | Hostname used for HTTP upload. |
| `xmpp_stack_conference_host` | Conference hostname for MUC. |
| `xmpp_stack_public_ip` | Public IP used by TURN. |
| `xmpp_stack_public_ipv6` | Optional public IPv6 used for `AAAA` records. |
| `xmpp_stack_enable_nginx` | Enables the optional `nginx` reverse-proxy layer. Default: `true`. |
| `xmpp_stack_admin_jid` | Administrative JID. |
| `xmpp_stack_certbot_email` | Contact email for Let's Encrypt. |
| `xmpp_stack_certbot_domains` | Domains passed to certificate issuance. |

### Shared Network Ports

| Variable | Description |
| --- | --- |
| `xmpp_stack_http_port` | Public HTTP port used for ACME and optional `nginx`. Default: `80`. |
| `xmpp_stack_https_port` | Public HTTPS port for `nginx` when enabled. Default: `443`. |
| `xmpp_stack_c2s_port` | XMPP client-to-server port. Default: `5222`. |
| `xmpp_stack_s2s_port` | XMPP server-to-server port. Default: `5269`. |
| `xmpp_stack_ejabberd_http_port` | ejabberd HTTP/WebSocket/upload port. Public when `nginx` is disabled. Default: `5443`. |
| `xmpp_stack_turn_port` | TURN/STUN TCP/UDP port. Default: `3478`. |
| `xmpp_stack_turn_tls_port` | TURN over TLS port. Default: `5349`. |
| `xmpp_stack_turn_relay_min_port` | TURN relay range start. Default: `49152`. |
| `xmpp_stack_turn_relay_max_port` | TURN relay range end. Default: `49200`. |

### Shared Paths And Host Settings

| Variable | Description |
| --- | --- |
| `xmpp_stack_packages` | Packages installed on the target host. Default: `podman`. |
| `xmpp_stack_app_root` | Runtime root for rendered configs and build contexts. |
| `xmpp_stack_data_root` | Base directory for persistent data. |
| `xmpp_stack_postgres_data_dir` | PostgreSQL data path. |
| `xmpp_stack_upload_dir` | ejabberd upload path. |
| `xmpp_stack_letsencrypt_dir` | Certificate storage path. |
| `xmpp_stack_backups_dir` | Backup destination path. |

### Shared Secrets And Derived Values

| Variable | Description |
| --- | --- |
| `xmpp_stack_secrets` | Secret dictionary loaded from vault. |
| `xmpp_stack_dns_secrets` | Secret dictionary for the optional `dns` role. |
| `xmpp_stack_postgres_password` | Derived from `xmpp_stack_secrets.postgres_password`. |
| `xmpp_stack_turn_static_auth_secret` | Derived from `xmpp_stack_secrets.turn_static_auth_secret`. |
| `xmpp_stack_turn_realm` | TURN realm. Default: `xmpp_stack_domain`. |

### Operational Toggles

| Variable | Description |
| --- | --- |
| `xmpp_stack_issue_certificates` | Allows certificate bootstrap in the `certbot` role. |
| `xmpp_stack_build_images` | Enables local image rebuilds for `ejabberd` and `coturn`. |
| `xmpp_stack_manage_dns` | Enables the optional `dns` role. |
| `xmpp_stack_firewall_enabled` | Enables the `firewall` role. Default: `true`. |
| `xmpp_stack_firewall_ssh_port` | SSH port allowed by the firewall. Default: `22`. |
| `xmpp_stack_firewall_ssh_allowed_ipv4` | Optional IPv4 allowlist for SSH. |
| `xmpp_stack_firewall_ssh_allowed_ipv6` | Optional IPv6 allowlist for SSH. |
| `xmpp_stack_firewall_additional_tcp_ports` | Extra TCP ports to allow through the firewall. |
| `xmpp_stack_firewall_additional_udp_ports` | Extra UDP ports to allow through the firewall. |
| `xmpp_stack_log_level` | ejabberd log verbosity. |
| `xmpp_stack_enable_federation` | Exposes `5269/tcp` and enables ejabberd server-to-server handling. |
| `xmpp_stack_enable_invites` | Enables ejabberd `mod_invites` for invite-based onboarding. |
| `xmpp_stack_invites_landing_page` | Landing page mode passed to `mod_invites`. Default: `none`. |
| `xmpp_stack_invites_max` | Per-user invite cap used by `mod_invites`. |
| `xmpp_stack_invites_token_expire_seconds` | Invite token lifetime in seconds. |
| `xmpp_stack_c2s_shaper_rate` | Rate limit used for the default client-to-server shaper. |
| `xmpp_stack_s2s_shaper_rate` | Rate limit used for the default server-to-server shaper. |
| `xmpp_stack_max_user_sessions` | Session cap used by ejabberd shaper rules. |
| `xmpp_stack_max_user_offline_messages` | Offline message cap used by ejabberd shaper rules. |

### DNS Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_dns_provider` | DNS provider for the optional `dns` role. Current practical value: `cloudflare`. |
| `xmpp_stack_dns_zone` | DNS zone managed by the provider. Default: `xmpp_stack_domain`. |
| `xmpp_stack_dns_regru_api_endpoint` | Base URL for the Reg.ru API backend. |
| `xmpp_stack_dns_ttl` | Default TTL for managed DNS records. |
| `xmpp_stack_dns_proxied` | Whether the apex record should be proxied on Cloudflare. Default: `false`. |
| `xmpp_stack_dns_manage_srv` | Enables optional SRV record management. |
| `xmpp_stack_dns_manage_txt` | Enables optional TXT record management. |
| `xmpp_stack_websocket_url` | URL published in `_xmppconnect` for WebSocket discovery. |
| `xmpp_stack_http_upload_url` | Public HTTP upload URL used by ejabberd. |
| `xmpp_stack_http_upload_max_size` | Maximum HTTP upload size in bytes used by ejabberd and nginx. Default: `52428800`. |
| `xmpp_stack_dns_records` | Declarative list of DNS records to manage. |
| `xmpp_stack_dns_srv_records` | Declarative list of SRV records to manage when enabled. |
| `xmpp_stack_dns_txt_records` | Declarative list of TXT records to manage when enabled. |

## Notes

- This role owns the shared defaults in `defaults/main.yml`.
- Other service roles depend on variables defined here even if they do not
  declare their own defaults.
