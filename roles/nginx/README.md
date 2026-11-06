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

- This role provides the HTTPS entrypoint for ejabberd HTTP endpoints and,
  when enabled, the static [root site](../../docs/root-site.md).
- Disable it when you prefer to expose ejabberd HTTP endpoints directly on `xmpp_stack_ejabberd_http_port`.
- It is enabled before bootstrap, but only started immediately when
  certificates already exist.
- Shared stack variables such as domain, upload host, paths, and certificate
  storage are documented in [common](../common/README.md).

## Shared HTTP Configuration

The template `templates/containers/nginx/nginx.conf.j2` sets these directives
inside `http`:

```nginx
server_names_hash_bucket_size 64;
include /etc/nginx/mime.types;
default_type application/octet-stream;
```

The bucket size fixes the startup failure `could not build server_names_hash`
seen with the configured hostnames and a bucket size of 32. MIME mappings ensure
that static assets such as CSS are served with the correct content type.
Keep these settings in the template: Ansible overwrites the runtime configuration.

Configuration changes currently notify a service restart; the role does not
run `nginx -t` automatically. See [operations](../../docs/operations.md#nginx)
for validation, recovery, and reload commands.
