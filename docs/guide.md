# Deployment Guide

## Overview

This repository deploys a private XMPP stack with Ansible. The target runtime is
Podman Quadlet on a Linux host.

Managed services:
- `ejabberd` for XMPP, multi-device state, MUC, upload, push, and WebSocket
- `postgres` for message archive and persistent server-side data
- `coturn` for STUN/TURN and more reliable calls across NAT
- optional `nginx` for HTTP/HTTPS entrypoints on `80/443`
- `certbot` for initial certificate issuance and renewals
- `firewall` for nftables-based host filtering
- optional `dns` automation for provider-managed zones

The repository follows a collection-style layout, but is operated locally as a
single project. Instead of one monolithic role, it is now split into service
roles with a clear execution order.

## Repository Layout

- `galaxy.yml`
  Collection metadata.
- `meta/runtime.yml`
  Ansible runtime requirements.
- `roles/common/`
  Shared validation, host packages, directories, `.env`, Podman network, and helper scripts.
- `roles/postgres/`
  PostgreSQL Quadlet and service lifecycle.
- `roles/dns/`
  Optional DNS management role. Cloudflare and Reg.ru are implemented.
- `roles/firewall/`
  nftables-based host firewall role aligned with the stack ports.
- `roles/ejabberd/`
  ejabberd config, local image build, Quadlet, and service lifecycle.
- `roles/coturn/`
  coturn config, local image build, Quadlet, and service lifecycle.
- `roles/nginx/`
  Optional nginx config, Quadlet, and service lifecycle.
- `roles/certbot/`
  certificate bootstrap, renewal scripts, and renewal timer.
- `playbooks/`
  Entry points for bootstrap and normal deploy.
- `inventories/production/`
  Example production inventory.
- `docs/`
  Operational and supporting documentation.
- `ci/`
  Minimal CI validation placeholders.

## Execution Order

The playbooks apply roles in this order:
1. `common`
2. `firewall`
3. `dns`
4. `postgres`
5. `ejabberd`
6. `coturn`
7. optional `nginx`
8. `certbot`

This order matters:
- shared directories and Quadlet network must exist before service roles run
- the firewall is applied before services start so exposed ports stay explicit
- DNS should be in place before the first certificate bootstrap if you enable automated DNS management
- `postgres` must be available before ejabberd starts
- `ejabberd`, `coturn`, and optional `nginx` install their units before the first certificate bootstrap
- `certbot` issues the first certificate and then restarts TLS-dependent services

## Requirements

Recommended target host:
- `2 vCPU`
- `4 GB RAM`
- `40+ GB SSD/NVMe`
- public `IPv4`
- `IPv6` if available

Recommended control host:
- `ansible-core >= 2.15`
- SSH access to the target host
- ability to use `become`
- optional: `pre-commit` for local validation before pushing changes

Target host assumptions:
- systemd available
- package manager can install `podman`
- inbound firewall can expose the required ports

## Runtime Layout On Target Hosts

The roles create and use:
- `/opt/xmpp-infra`
  Rendered runtime configuration, scripts, and container build contexts.
- `/srv/xmpp/postgres`
  PostgreSQL data.
- `/srv/xmpp/ejabberd/upload`
  Uploaded files.
- `/srv/xmpp/letsencrypt`
  Certificate state.
- `/srv/xmpp/backups`
  Backup destination.

## Inventory Layout

Use the production inventory structure:

- `inventories/production/hosts.yml`
- `inventories/production/group_vars/xmpp_hosts/main.yml`
- `inventories/production/group_vars/xmpp_hosts/vault.yml`

Keep secrets in `vault.yml` encrypted with `ansible-vault`.

### Non-sensitive variables

Typical values in `main.yml`:
- `xmpp_stack_domain`
- `xmpp_stack_upload_host`
- `xmpp_stack_conference_host`
- `xmpp_stack_public_ip`
- `xmpp_stack_public_ipv6`
- `xmpp_stack_admin_jid`
- `xmpp_stack_certbot_email`
- `xmpp_stack_certbot_domains`
- `xmpp_stack_manage_dns`
- `xmpp_stack_dns_provider`
- `xmpp_stack_dns_zone`

### Sensitive variables

Typical values in `vault.yml`:
- `xmpp_stack_secrets.postgres_password`
- `xmpp_stack_secrets.turn_static_auth_secret`
- `xmpp_stack_dns_secrets.cloudflare_api_token`
- `xmpp_stack_dns_secrets.regru_username`
- `xmpp_stack_dns_secrets.regru_password`

Provider-specific inventory examples:
- `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.main.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.vault.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/regru.main.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/regru.vault.yml.example`

## Important Variables

Core variables:
- `xmpp_stack_domain`
  Main XMPP domain.
- `xmpp_stack_upload_host`
  Hostname used for HTTP upload.
- `xmpp_stack_conference_host`
  Conference domain used by MUC.
- `xmpp_stack_public_ip`
  Public IP used by TURN.
- `xmpp_stack_public_ipv6`
  Optional IPv6 used for `AAAA` records.
- `xmpp_stack_admin_jid`
  Administrative JID.
- `xmpp_stack_certbot_email`
  Email passed to Let's Encrypt.
- `xmpp_stack_certbot_domains`
  Domain list used during certificate issuance.
  In the expanded profile this should usually include the main XMPP domain,
  `upload`, and `conference`.

Secrets:
- `xmpp_stack_secrets.postgres_password`
- `xmpp_stack_secrets.turn_static_auth_secret`

Operational toggles:
- `xmpp_stack_issue_certificates`
  Whether the certbot role may bootstrap the first certificate.
- `xmpp_stack_expand_certificates`
  Whether the certbot role may expand an existing certificate when the effective domain set grows.
- `xmpp_stack_build_images`
  Whether ejabberd and coturn roles build local images.
- `xmpp_stack_manage_dns`
  Whether the optional DNS role should manage records.
- `xmpp_stack_log_level`
  ejabberd log level.
- `xmpp_stack_enable_nginx`
  Enables the optional reverse-proxy layer on `80/443`.
- `xmpp_stack_enable_federation`
  Enables the `5269/tcp` listener and ejabberd s2s modules.
- `xmpp_stack_enable_invites`
  Enables ejabberd `mod_invites` for invite-based onboarding.

ejabberd tuning variables:
- `xmpp_stack_invites_landing_page`
- `xmpp_stack_invites_max`
- `xmpp_stack_invites_token_expire_seconds`
- `xmpp_stack_c2s_shaper_rate`
- `xmpp_stack_s2s_shaper_rate`
- `xmpp_stack_max_user_sessions`
- `xmpp_stack_max_user_offline_messages`

DNS variables:
- `xmpp_stack_dns_provider`
- `xmpp_stack_dns_zone`
- `xmpp_stack_dns_ttl`
- `xmpp_stack_dns_proxied`
- `xmpp_stack_dns_manage_srv`
- `xmpp_stack_dns_manage_txt`
- `xmpp_stack_websocket_url`
- `xmpp_stack_http_upload_url`
- `xmpp_stack_dns_records`
- `xmpp_stack_dns_srv_records`
- `xmpp_stack_dns_txt_records`

Paths:
- `xmpp_stack_app_root`
- `xmpp_stack_data_root`
- `xmpp_stack_postgres_data_dir`
- `xmpp_stack_upload_dir`
- `xmpp_stack_letsencrypt_dir`
- `xmpp_stack_backups_dir`

## Playbooks

- `playbooks/bootstrap.yml`
  First-run path. Allows initial certificate issuance.
- `playbooks/deploy.yml`
  Regular deployment path. Intended for updates after the system already exists.
  This path keeps first-run certificate bootstrap disabled, but may still expand
  an existing certificate SAN set when `xmpp_stack_expand_certificates: true`.
- `playbooks/site.yml`
  All-in-one entry point.

### Recommended usage

Initial bring-up:

```sh
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass
```

Normal updates:

```sh
ansible-playbook playbooks/deploy.yml --ask-vault-pass
```

## Tags

Shared tags:
- `validate`
- `packages`
- `directories`
- `config`
- `images`
- `bootstrap`
- `certificates`
- `services`

Role tags:
- `common`
- `firewall`
- `dns`
- `postgres`
- `ejabberd`
- `coturn`
- `nginx`
- `certbot`

Example targeted runs:

```sh
ansible-playbook playbooks/deploy.yml --ask-vault-pass --tags config,ejabberd
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass --tags certificates,certbot
```

## Deployment Flow

The playbooks perform these high-level stages:

1. Validate required variables.
2. Install host packages.
3. Create runtime directories.
4. Render shared runtime assets and the Podman network.
5. Apply nftables firewall rules for the host.
6. Optionally manage DNS records through the configured provider.
7. Render Quadlet files and service-specific configs.
8. Build local images for `ejabberd` and `coturn` if needed.
9. Start `postgres` and enable TLS-dependent services.
10. If the first certificate is missing and allowed, bootstrap it with `certbot`.
11. Enable the renewal timer.

The image build logic is hash-based, so rebuilds only occur when relevant input
files change.

TLS-dependent services are enabled before bootstrap but only started
immediately when certificates already exist. On the first run, the certbot
bootstrap script issues certificates and restarts `ejabberd`, `coturn`, and
`nginx` afterwards when the proxy layer is enabled.

## DNS

See also:
- [dns.md](./dns.md)
- [DNS_PROVIDERS.md](./DNS_PROVIDERS.md)

At minimum you usually need:
- `A/AAAA` for `xmpp_stack_domain`
- `A/AAAA` for `xmpp_stack_upload_host`
- `A/AAAA` for `xmpp_stack_conference_host`

Optional SRV records can improve interoperability:
- `_xmpp-client._tcp.<domain>`
- `_xmpp-server._tcp.<domain>`

If you enable the `dns` role with Cloudflare, these records can be managed from
Ansible. Cloudflare support is implemented through
`community.general.cloudflare_dns`, so the control host needs the
`community.general` collection installed.

## Ports

See also:
- [ports.md](./ports.md)

Commonly exposed ports:
- `5222/tcp` for XMPP client-to-server
- `5269/tcp` for XMPP server-to-server
- `5443/tcp` for ejabberd HTTP upload and WebSocket, public only when `nginx` is disabled
- `3478/tcp,udp` for STUN/TURN
- `5349/tcp` for TURN over TLS
- `49152-49200/udp` for TURN relay
- `80/tcp` for `certbot` standalone ACME
- `443/tcp` for `nginx` when enabled

## Certificates

Certificates are stored under:
- `/srv/xmpp/letsencrypt`

The initial certificate is issued by:
- `scripts/certbot-bootstrap.sh` rendered by the `certbot` role

Renewals are handled through:
- `certbot-renew.service`
- `certbot-renew.timer`

During bootstrap or renewal, the proxy service may be stopped briefly so
`certbot --standalone` can bind to port `80` when `nginx` is enabled.

## Operations

See also:
- [operations.md](./operations.md)
- [secrets.md](./secrets.md)

Recommended routine checks:
- confirm services are active
- verify backups are created
- check mobile push
- verify upload works
- test a real external call periodically

## Troubleshooting

If the playbook fails early:
- confirm `vault.yml` is present and decrypted via `--ask-vault-pass`
- confirm the target host is reachable over SSH
- confirm `become` works

If certificate bootstrap fails:
- confirm DNS already points at the target host
- confirm port `80/tcp` is reachable from the internet
- confirm no other service is holding port `80`

If calls fail:
- confirm `xmpp_stack_public_ip` is correct
- confirm TURN ports are open
