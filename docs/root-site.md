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

Then apply the infrastructure changes:

```bash
ansible-playbook playbooks/deploy.yml -i inventories/production/hosts.yml --ask-vault-pass --ask-become-pass --tags dns,common,certbot,nginx
```

Finally verify:

```bash
curl -Ik https://example.com
curl -Ik https://www.example.com
```
