# ejabberd

Manages:
- ejabberd container build context
- local ejabberd image rebuilds
- ejabberd Quadlet unit
- conditional service start after certificate bootstrap

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_ejabberd_image` | Local ejabberd image tag. |
| `xmpp_stack_build_images` | Whether the role should rebuild the local ejabberd image. |
| `xmpp_stack_log_level` | ejabberd log verbosity. |
| `xmpp_stack_postgres_db` | Database name used by ejabberd. |
| `xmpp_stack_postgres_user` | Database user used by ejabberd. |
| `xmpp_stack_postgres_password` | Database password used by ejabberd. |
| `xmpp_stack_upload_dir` | Host path mounted for HTTP upload storage. |

## Notes

- This role expects `postgres` to be available before the service is started.
- Image rebuilds are hash-based and only happen when the Containerfile or
  rendered `ejabberd.yml` changes.
- `mod_invites` can be enabled declaratively for invite-based onboarding
  without enabling public registration.
- Federation on `5269/tcp`, local API permissions, and default shaper rules are
  controlled through shared variables from [common](../common/README.md).
- Shared stack variables such as domain, admin JID, paths, and certificate
  settings are documented in [common](../common/README.md).
