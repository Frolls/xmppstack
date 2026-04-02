# Secrets

Do not store real secrets in git.

Use the inventory structure:
- `inventories/production/group_vars/xmpp_hosts/main.yml`
- `inventories/production/group_vars/xmpp_hosts/vault.yml`

Recommended secrets:
- PostgreSQL password
- TURN static auth secret
- deploy credentials, if your CI/CD needs them

Recommended storage:
- `ansible-vault` for inventory secrets
- CI secret variables for deploy access only
