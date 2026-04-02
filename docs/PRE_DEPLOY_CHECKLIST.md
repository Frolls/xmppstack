# Pre-Deploy Checklist

Use this checklist before the first real deployment or before moving the stack
to a new server.

## Inventory And Secrets

- `inventories/production/hosts.yml` points to the intended target host.
- `inventories/production/group_vars/xmpp_hosts/main.yml` is filled out.
- `inventories/production/group_vars/xmpp_hosts/vault.yml` is encrypted with `ansible-vault`.
- `xmpp_stack_domain`, `xmpp_stack_upload_host`, and `xmpp_stack_conference_host` are correct.
- `xmpp_stack_admin_jid` belongs to the intended domain.
- `xmpp_stack_public_ip` is the real public IPv4 of the server.
- `xmpp_stack_public_ipv6` is set if the host is reachable over IPv6.
- `xmpp_stack_firewall_ssh_port` matches the real SSH port on the server.
- `xmpp_stack_enable_nginx` matches the intended public HTTP topology.
- `xmpp_stack_certbot_domains` includes every TLS hostname you expect to serve.

## DNS

- The zone is delegated and writable through the chosen provider.
- If `xmpp_stack_manage_dns: true`, provider credentials are present in vault.
- If `xmpp_stack_manage_dns: false`, required records already exist manually.
- `A` and optional `AAAA` records resolve to the correct server.
- `_xmpp-client._tcp` points to the correct c2s port.
- `_xmpp-server._tcp` is published only if federation is intended.
- `_xmppconnect` TXT matches the WebSocket URL if you keep TXT discovery enabled.

## Firewall And Network

- The VPS provider firewall or security group allows the same ports as the host firewall.
- SSH access is confirmed before enabling nftables.
- Ports expected to be public are known:
  - `22/tcp`
  - `80/tcp`
  - `443/tcp` when `xmpp_stack_enable_nginx: true`
  - `5222/tcp`
  - `5269/tcp` when federation is enabled
  - `5443/tcp` when `xmpp_stack_enable_nginx: false`
  - `3478/tcp,udp`
  - `5349/tcp`
  - `49152-49200/udp`
- Internal-only ports are not expected to be public:
  - `5443/tcp` when `xmpp_stack_enable_nginx: true`
  - PostgreSQL ports

## Certificates

- `xmpp_stack_certbot_email` is valid.
- `xmpp_stack_certbot_domains` includes at least the main domain and `upload`.
- `conference` is included if you want the expanded profile and remote compatibility.
- Port `80/tcp` is reachable for the initial ACME bootstrap.

## TURN And Media

- `xmpp_stack_turn_static_auth_secret` is set.
- `xmpp_stack_turn_realm` matches the intended XMPP domain unless you have a reason to override it.
- `xmpp_stack_public_ip` is not a private or NATed internal address.
- Relay port range is acceptable for the host and provider firewall.

## Control Host

- `ansible-core` is installed.
- `ansible-playbook` works locally.
- `community.general` is installed when DNS automation is enabled.
- `ansible-lint` and optional `pre-commit` are available if you use local validation.

## Dry Validation

- Run `ansible-playbook playbooks/bootstrap.yml --syntax-check`.
- Run `ansible-lint`.
- Review rendered inventory values one more time before the first bootstrap.
