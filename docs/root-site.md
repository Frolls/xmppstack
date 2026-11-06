# Root Site

This repository can serve an additional static site from the same `nginx`
instance that fronts the XMPP stack. The collection is responsible only for:

- provisioning the host directory for built static files
- mounting that directory into the shared `nginx` container
- rendering the extra `nginx` vhost for the root domain
- keeping DNS and TLS settings aligned with the extra hostname

The site source itself can live in a separate repository.

## Inventory

Set these variables in your inventory:

```yaml
xmpp_stack_site_enabled: true
xmpp_stack_site_domain: example.com
xmpp_stack_site_www_domain: www.example.com
```

The built static files are served from:

```yaml
xmpp_stack_site_root: /opt/xmpp-infra/site/root
```

## Certificates

Keep `xmpp_stack_certbot_domains` focused on the base stack names, for example:

```yaml
xmpp_stack_certbot_domains:
  - xmpp.example.com
  - upload.xmpp.example.com
  - conference.xmpp.example.com
```

When both `xmpp_stack_enable_nginx: true` and `xmpp_stack_site_enabled: true`
are set, the collection automatically adds `xmpp_stack_site_domain` and
`xmpp_stack_site_www_domain` to the effective certificate domain list.

During deploy the collection compares the current certificate SAN list with the
effective domain set. If new domains were added, the certificate is re-issued
automatically with the same cert name based on `xmpp_stack_domain`.
This SAN expansion path is controlled by `xmpp_stack_expand_certificates`
and is enabled by default.

The shared nginx configuration reads certificates from:

```text
/etc/letsencrypt/live/{{ xmpp_stack_domain }}/
```

## DNS

If you manage DNS through this repository, make sure the authoritative zone is
the root zone of the site and add records for the site hostnames. When the
inventory already overrides `xmpp_stack_dns_records`, include at least:

```yaml
- record: "@"
  type: A
  value: <public-ip>
  ttl: 300
- record: "www"
  type: A
  value: <public-ip>
  ttl: 300
```

## Site Deployment

Build the site in its own repository with any static site generator you prefer.
The result should be a directory with ready-to-serve static files.

Copy the built files to the target host:

```bash
rsync -av --delete <build-dir>/ <user>@<host>:/tmp/root-site/
ssh <user>@<host> 'sudo mkdir -p /opt/xmpp-infra/site/root && sudo rsync -av --delete /tmp/root-site/ /opt/xmpp-infra/site/root/'
```

For initial setup or changes to domains, certificates, or site paths, apply the
infrastructure changes:

```bash
ansible-playbook playbooks/deploy.yml -i inventories/production/hosts.yml --ask-vault-pass --ask-become-pass --tags dns,common,certbot,nginx
```

Once infrastructure is configured, deploying new static files does not require
redeploying the stack or restarting nginx.

For changes limited to the nginx template, apply only that role:

```bash
ansible-playbook playbooks/deploy.yml -i inventories/production/hosts.yml --ask-vault-pass --ask-become-pass --tags nginx
```

Shared variable validation still runs. This command can restart the shared nginx
service, affecting both the site and XMPP HTTPS endpoints; it does not redeploy
PostgreSQL, ejabberd, or coturn. The role currently has no automatic `nginx -t`
validation before restart. See [operations](operations.md#nginx) for manual
configuration checks and recovery. If the corrected configuration is already
running, no redeploy is needed solely to record the same fix in Git.

Finally verify from an external network, with certificate validation enabled:

```bash
curl --fail --show-error -I https://example.com/
curl --fail --show-error -I https://www.example.com/
```

Expect `200` for the root page and a redirect from `www` to the root domain.
Also request an actual CSS URL referenced by the HTML and check for `200` and
`Content-Type: text/css`. A running service or an existing `index.html` alone
does not confirm that the site works.

The shared [nginx template](../roles/nginx/templates/containers/nginx/nginx.conf.j2)
includes MIME mappings and `server_names_hash_bucket_size 64` to preserve these
settings on future deployments.
