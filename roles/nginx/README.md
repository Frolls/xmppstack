# nginx

Renders the nginx configuration, installs its Quadlet unit, and starts the
service once certificates are present.

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_nginx_image` | nginx container image. Default: `docker.io/library/nginx:1.28-alpine`. |

## Notes

- This role acts only as the HTTPS entrypoint for ejabberd HTTP endpoints.
- It is enabled before bootstrap, but only started immediately when
  certificates already exist.
- Shared stack variables such as domain, upload host, paths, and certificate
  storage are documented in [common](../common/README.md).
