# firewall

Manages:
- nftables package installation
- rendered host firewall rules
- nftables service enablement

## Variables

### Role Variables

| Variable | Description |
| --- | --- |
| `xmpp_stack_firewall_enabled` | Enables or disables this role. Default: `true`. |
| `xmpp_stack_firewall_ssh_port` | SSH port allowed by the firewall. Default: `22`. |
| `xmpp_stack_firewall_ssh_allowed_ipv4` | Optional IPv4 allowlist for SSH. Empty means allow from anywhere. |
| `xmpp_stack_firewall_ssh_allowed_ipv6` | Optional IPv6 allowlist for SSH. Empty means allow from anywhere. |
| `xmpp_stack_firewall_additional_tcp_ports` | Extra TCP ports to allow. |
| `xmpp_stack_firewall_additional_udp_ports` | Extra UDP ports to allow. |

## Notes

- The role uses `nftables` and keeps allowed ports aligned with the shared
  stack variables from [common](../common/README.md).
- `5443/tcp` is intentionally not exposed publicly because it is an internal
  ejabberd HTTP port behind `nginx`.
- If you use a non-default SSH port, set `xmpp_stack_firewall_ssh_port` before
  applying the role.
