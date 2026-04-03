# nginx

Renders the nginx configuration, installs its Quadlet unit, and starts the
service once certificates are present.

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_enable_nginx` | Enables this optional role from the playbook level. Default: `true`. |
| `xmpp_stack_nginx_image` | nginx container image. Default: `docker.io/library/nginx:1.28-alpine`. |
| `xmpp_stack_http_upload_max_size` | Maximum request body size nginx should accept for ejabberd uploads. |

## Notes

- This role acts only as the HTTPS entrypoint for ejabberd HTTP endpoints.
- Disable it when you prefer to expose ejabberd HTTP endpoints directly on `xmpp_stack_ejabberd_http_port`.
- It is enabled before bootstrap, but only started immediately when
  certificates already exist.
- Shared stack variables such as domain, upload host, paths, and certificate
  storage are documented in [common](../common/README.md).
