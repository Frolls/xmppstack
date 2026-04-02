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
| `xmpp_stack_certbot_email` | Contact email passed to Let's Encrypt. |
| `xmpp_stack_certbot_domains` | Domain list used during initial issuance. |
| `xmpp_stack_certbot_renew_schedule` | systemd timer schedule for renewals. |

## Notes

- On first bootstrap this role starts only the minimal prerequisite services,
  requests the certificate, and then restarts TLS-dependent services.
- Renewals are handled through rendered `systemd` units rather than cron.
- For the expanded DNS/XMPP profile, include at least the main domain,
  `upload` host, and usually the `conference` host in
  `xmpp_stack_certbot_domains`.
- Shared stack variables such as domain, paths, and certificate storage are
  documented in [common](../common/README.md).
- Bootstrap assumes the rest of the service roles has already rendered their
  units and runtime paths.
