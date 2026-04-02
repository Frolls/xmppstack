# XMPP Infra

This repository is structured as a collection-style Ansible project for
deploying a private XMPP stack with:
- `ejabberd`
- `postgres`
- `coturn`
- `nginx`
- `certbot`
- `firewall`
- optional `dns`

The supported deployment path is Ansible only.

Detailed documentation:
- [Deployment Guide](./docs/guide.md)
- [Руководство По Развёртыванию](./docs/guide.ru.md)
- [Pre-Deploy Checklist](./docs/PRE_DEPLOY_CHECKLIST.md)
- [Чеклист Перед Деплоем](./docs/PRE_DEPLOY_CHECKLIST.ru.md)
- [Post-Deploy Checklist](./docs/POST_DEPLOY_CHECKLIST.md)
- [Чеклист После Деплоя](./docs/POST_DEPLOY_CHECKLIST.ru.md)
- [Test Plan](./docs/TEST_PLAN.md)
- [План Тестирования](./docs/TEST_PLAN.ru.md)

## Layout

- [galaxy.yml](./galaxy.yml)
- [meta/runtime.yml](./meta/runtime.yml)
- [roles/common](./roles/common)
- [roles/postgres](./roles/postgres)
- [roles/dns](./roles/dns)
- [roles/firewall](./roles/firewall)
- [roles/ejabberd](./roles/ejabberd)
- [roles/coturn](./roles/coturn)
- [roles/nginx](./roles/nginx)
- [roles/certbot](./roles/certbot)
- [playbooks](./playbooks)
- [inventories](./inventories)
- [docs](./docs)

## Quick Start

```sh
mkdir -p inventories/production/group_vars/xmpp_hosts
cp inventories/production/group_vars/xmpp_hosts/main.yml.example inventories/production/group_vars/xmpp_hosts/main.yml
cp inventories/production/group_vars/xmpp_hosts/vault.yml.example inventories/production/group_vars/xmpp_hosts/vault.yml
$EDITOR inventories/production/hosts.yml
$EDITOR inventories/production/group_vars/xmpp_hosts/main.yml
$EDITOR inventories/production/group_vars/xmpp_hosts/vault.yml
ansible-vault encrypt inventories/production/group_vars/xmpp_hosts/vault.yml
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass
ansible-playbook playbooks/deploy.yml --ask-vault-pass
```

This quick start is intentionally short. For architecture, variables, bootstrap
flow, operations, and troubleshooting, use the detailed guides in `docs/`.

## Validation

Static validation in CI uses:
- `ansible-playbook --syntax-check`
- `ansible-lint`
- role metadata checks for `meta/main.yml` and `meta/argument_specs.yml`

Required control-host dependency examples are described in:
- [requirements.yml](./requirements.yml)
- [.ansible-lint](./.ansible-lint)
- [.pre-commit-config.yaml](./.pre-commit-config.yaml)

Optional local validation with `pre-commit`:

```sh
python -m pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## Playbooks

- [bootstrap.yml](./playbooks/bootstrap.yml): first run, including initial certificate issuance
- [deploy.yml](./playbooks/deploy.yml): regular deployments and updates
- [site.yml](./playbooks/site.yml): all-in-one run

## Vault Layout

- `inventories/production/group_vars/xmpp_hosts/main.yml`
  Non-sensitive variables.
- `inventories/production/group_vars/xmpp_hosts/vault.yml`
  Sensitive variables encrypted with `ansible-vault`.

## Tags

- `validate`
- `packages`
- `directories`
- `config`
- `images`
- `bootstrap`
- `certificates`
- `services`
- `common`
- `firewall`
- `dns`
- `postgres`
- `ejabberd`
- `coturn`
- `nginx`
- `certbot`

## Runtime Layout

- app root: `/opt/xmpp-infra`
- postgres data: `/srv/xmpp/postgres`
- ejabberd uploads: `/srv/xmpp/ejabberd/upload`
- letsencrypt state: `/srv/xmpp/letsencrypt`
- backups: `/srv/xmpp/backups`

Suggested VPS baseline:
- `2 vCPU`
- `4 GB RAM`
- `40+ GB SSD/NVMe`
- public `IPv4`
- `IPv6` if available

Notes:
- Do not commit real secrets.
- Keep database, uploads, and letsencrypt state in persistent volumes.
- Test TURN from an external network before migration.
- If you enable DNS automation, install `community.general` on the control host.
