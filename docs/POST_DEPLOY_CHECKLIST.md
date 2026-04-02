# Post-Deploy Checklist

Use this checklist right after the first successful deployment.

## Service State

- `postgres.service` is enabled and running.
- `ejabberd.service` is enabled and running.
- `coturn.service` is enabled and running.
- `nginx.service` is enabled and running.
- `nftables.service` is enabled and running.
- `certbot-renew.timer` is enabled and active.

## TLS

- A certificate exists for the main XMPP domain.
- A certificate exists for `upload` if it is served separately.
- `conference` is covered if included in `xmpp_stack_certbot_domains`.
- HTTPS works on the public endpoint.

## Network Checks

- `80/tcp` redirects to HTTPS as expected.
- `443/tcp` responds correctly.
- `5222/tcp` is reachable from an external network.
- `5269/tcp` is reachable if federation is enabled.
- `3478/tcp,udp` and `5349/tcp` are reachable for TURN.
- TURN relay UDP range is not blocked by provider-side firewalling.
- `5443/tcp` is not exposed publicly unless you explicitly intended that.

## XMPP Functionality

- Login works from a desktop client.
- Login works from a mobile client.
- One user can connect from multiple devices.
- Message archive works across devices.
- OMEMO session setup works.
- HTTP upload works.
- Room creation and join work.
- Push notifications arrive on mobile.

## Call Testing

- A direct call works between two clients on different networks.
- A call still works when at least one side is behind a restrictive NAT or mobile network.
- TURN is actually used when peer-to-peer connection is not possible.

## DNS Validation

- The main domain resolves correctly.
- `upload` resolves correctly.
- `conference` resolves correctly if used.
- `_xmpp-client._tcp` resolves correctly.
- `_xmpp-server._tcp` resolves correctly if federation is enabled.
- `_xmppconnect` TXT is correct if TXT discovery is enabled.

## Operations

- Backups can be created successfully.
- Upload storage is writable.
- PostgreSQL data persists across service restarts.
- Certificates survive service restarts.
- Logs are readable and do not show obvious bootstrap errors.

## Security Review

- SSH is still reachable after firewall activation.
- Only the intended public ports are visible from outside.
- `vault.yml` is still encrypted and was not copied into logs or plaintext artifacts.
