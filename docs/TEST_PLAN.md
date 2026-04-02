# Test Plan

Use this plan after the first deployment to validate the stack end to end on a
test server before calling it production-ready.

## 1. Host And Service Baseline

- Confirm the host is reachable over SSH.
- Confirm `nftables.service` is active.
- Confirm `postgres.service` is active.
- Confirm `ejabberd.service` is active.
- Confirm `coturn.service` is active.
- Confirm `nginx.service` is active when `xmpp_stack_enable_nginx: true`.
- Confirm `certbot-renew.timer` is active.

## 2. Public Port Validation

From an external network, verify:
- `80/tcp` answers and redirects to HTTPS when `xmpp_stack_enable_nginx: true`.
- `443/tcp` answers with a valid certificate when `xmpp_stack_enable_nginx: true`.
- `5222/tcp` is reachable.
- `5269/tcp` is reachable if federation is enabled.
- `5443/tcp` answers with a valid certificate when `xmpp_stack_enable_nginx: false`.
- `3478/tcp,udp` is reachable.
- `5349/tcp` is reachable.
- TURN relay UDP range is not blocked upstream.

Also verify that:
- `5443/tcp` is not publicly reachable when `xmpp_stack_enable_nginx: true`.
- PostgreSQL is not publicly reachable.

## 3. DNS Validation

Verify:
- apex `A` and optional `AAAA`
- `upload` `A` and optional `AAAA`
- `conference` `A` and optional `AAAA`
- `_xmpp-client._tcp`
- `_xmpp-server._tcp` if federation is enabled
- `_xmppconnect` TXT if TXT discovery is enabled
- STUN/TURN SRV records if you use the expanded DNS profile

## 4. TLS Validation

Verify:
- certificate subject/SANs include expected names
- `upload` is covered
- `conference` is covered if included in the profile
- certificate files are readable by the services that need them

## 5. XMPP Client Tests

With a desktop client:
- add the account
- log in successfully
- send and receive messages
- verify message archive history
- verify multi-device sync if a second client is connected

With a mobile client:
- log in successfully
- verify push delivery
- verify reconnect after backgrounding the app

## 6. OMEMO And Multi-Device

- establish OMEMO between two clients
- verify encrypted messaging works both ways
- verify a user with two devices receives synchronized history
- verify a new session does not break message flow

## 7. Rooms And Upload

- create a room
- join the room from another client
- verify room persistence
- send files through HTTP upload
- confirm uploaded files can be downloaded through the public HTTPS endpoint

## 8. Call And TURN Validation

- make a call between two clients on different networks
- test one client behind mobile data or restrictive NAT
- confirm the call still succeeds
- check service logs to confirm TURN was used when direct connectivity failed

## 9. Federation Validation

Only if federation is enabled:
- verify `_xmpp-server._tcp` resolves correctly
- test connectivity from another XMPP server or remote account
- confirm s2s TLS and dialback behavior are normal in logs

## 10. Failure Recovery

- restart `ejabberd.service` and verify clients reconnect
- restart `nginx.service` and verify HTTPS returns when `xmpp_stack_enable_nginx: true`
- restart `coturn.service` and re-test calls
- restart `postgres.service` and confirm ejabberd recovers
- reboot the host and verify all required services come back automatically

## 11. Backup And Persistence

- run the backup scripts
- verify the output files exist
- confirm PostgreSQL data persists after restart
- confirm upload files persist after restart

## 12. Final Review

The stack is a good production candidate when:
- all critical services start automatically
- XMPP login works on desktop and mobile
- OMEMO works
- rooms work
- upload works
- push works
- calls work in at least one TURN-required scenario
- DNS and certificates match the intended topology
- no unexpected ports are exposed
