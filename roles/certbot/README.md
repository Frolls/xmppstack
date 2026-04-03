# certbot

Handles:
- first certificate bootstrap
- renewal scripts
- systemd renewal timer

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_issue_certificates` | Controls whether first-run certificate bootstrap is allowed. |
| `xmpp_stack_expand_certificates` | Controls whether an existing certificate may be expanded when the effective domain list changes. |
| `xmpp_stack_certbot_email` | Contact email passed to Let's Encrypt. |
| `xmpp_stack_certbot_domains` | Base domain list used for certificate issuance. |
| `xmpp_stack_certbot_renew_schedule` | systemd timer schedule for renewals. |

## Notes

- On first bootstrap this role starts only the minimal prerequisite services,
  requests the certificate, and then restarts TLS-dependent services.
- If `xmpp_stack_enable_nginx: true`, helper scripts stop and restart
  `nginx` around standalone ACME runs.
- Renewals are handled through rendered `systemd` units rather than cron.
- For the expanded DNS/XMPP profile, include at least the main domain,
  `upload` host, and usually the `conference` host in
  `xmpp_stack_certbot_domains`.
- When `xmpp_stack_enable_nginx: true` and the optional root site is enabled,
  the site hostnames are appended automatically to the effective certificate
  domain set before bootstrap or SAN expansion.
- Shared stack variables such as domain, paths, and certificate storage are
  documented in [common](../common/README.md).
- Bootstrap assumes the rest of the service roles has already rendered their
  units and runtime paths.
