# Operations

Daily:
- check container health
- check push delivery
- review ejabberd logs
- confirm nftables rules still match the exposed services

Weekly:
- verify backups
- verify disk growth from uploads and archives
- test a real call from an external network

Before upgrades:
- back up PostgreSQL
- back up uploads
- keep previous image tags for rollback


## nginx

The shared nginx instance serves XMPP HTTPS endpoints and the optional static
root site. A failed nginx service can make both unavailable while ejabberd
continues to accept direct XMPP connections.

### Diagnose and validate

Run on the server:

```bash
sudo systemctl status nginx --no-pager
sudo journalctl -u nginx -n 100 --no-pager
```

For `could not build server_names_hash ... bucket_size: 32`, confirm that the
rendered `http` block contains `server_names_hash_bucket_size 64;`. This setting
is now tracked in the role template. Preserve template changes in Git as well as
any emergency runtime edits.

The following checks the rendered configuration without binding HTTP ports or
restarting services. It works even when the nginx service is stopped. Paths and
image below use the collection defaults; adjust them to your inventory.
Certificates and the image must already be present.

```bash
sudo podman run --rm --pull=never --network none --entrypoint nginx \
  -v /opt/xmpp-infra/containers/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /srv/xmpp/letsencrypt:/etc/letsencrypt:ro \
  -v /opt/xmpp-infra/site/root:/srv/root-site:ro \
  docker.io/library/nginx:1.28-alpine -t
```

Proceed only after `syntax is ok` and `test is successful`. The Ansible role
currently restarts nginx on configuration changes without performing this test
automatically; this manual check validates the rendered file, not a future
template change.

### Apply or recover

For configuration-only edits to a running container, validate and reload:

```bash
sudo podman exec nginx nginx -t && sudo podman exec nginx nginx -s reload
```

For a failed service, after fixing and validating the configuration:

```bash
sudo systemctl reset-failed nginx
sudo systemctl start nginx
sudo systemctl status nginx --no-pager
```

These commands do not explicitly restart ejabberd. The nginx unit depends on
ejabberd, so starting nginx can start ejabberd if it is inactive.

Check the [root page and static assets](root-site.md#site-deployment) over HTTPS
from outside the server. Monitor the public endpoint and alert on failures;
checking only `systemctl is-active` does not verify HTTP responses or TLS.
