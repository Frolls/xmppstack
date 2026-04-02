# postgres

Renders the PostgreSQL Quadlet unit and manages the `postgres.service` lifecycle.

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_postgres_image` | PostgreSQL container image. Default: `docker.io/library/postgres:16`. |
| `xmpp_stack_postgres_db` | Database name for ejabberd. |
| `xmpp_stack_postgres_user` | PostgreSQL user for ejabberd. |
| `xmpp_stack_postgres_password` | PostgreSQL password. Usually sourced from vault. |
| `xmpp_stack_postgres_data_dir` | Host path for persistent PostgreSQL data. |

## Notes

- This role expects the `common` role to run first, because it relies on the
  shared runtime directories and `.env`.
- It does not bootstrap schema manually; ejabberd handles schema updates on its
  side.
- Shared stack variables are documented in [common](../common/README.md).
