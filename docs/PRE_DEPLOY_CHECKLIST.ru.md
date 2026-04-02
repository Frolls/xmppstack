# Чеклист Перед Деплоем

Используйте этот чеклист перед первым боевым развёртыванием или перед
переносом стека на новый сервер.

## Inventory И Секреты

- `inventories/production/hosts.yml` указывает на нужный сервер.
- `inventories/production/group_vars/xmpp_hosts/main.yml` заполнен.
- `inventories/production/group_vars/xmpp_hosts/vault.yml` зашифрован через `ansible-vault`.
- `xmpp_stack_domain`, `xmpp_stack_upload_host` и `xmpp_stack_conference_host` заданы корректно.
- `xmpp_stack_admin_jid` относится к нужному домену.
- `xmpp_stack_public_ip` это реальный публичный IPv4 сервера.
- `xmpp_stack_public_ipv6` задан, если сервер доступен по IPv6.
- `xmpp_stack_firewall_ssh_port` совпадает с реальным SSH-портом на сервере.
- `xmpp_stack_enable_nginx` соответствует выбранной публичной HTTP-схеме.
- `xmpp_stack_certbot_domains` включает все hostname, для которых нужен TLS.

## DNS

- Зона делегирована и доступна на запись через выбранного провайдера.
- Если `xmpp_stack_manage_dns: true`, креды провайдера есть в vault.
- Если `xmpp_stack_manage_dns: false`, нужные записи уже заведены вручную.
- `A` и, при необходимости, `AAAA` указывают на правильный сервер.
- `_xmpp-client._tcp` указывает на правильный c2s-порт.
- `_xmpp-server._tcp` опубликован только если federation действительно нужна.
- `_xmppconnect` TXT совпадает с WebSocket URL, если вы оставляете TXT-discovery включённым.

## Firewall И Сеть

- Firewall/SG у VPS-провайдера разрешает те же порты, что и host firewall.
- SSH-доступ проверен до включения nftables.
- Публичные порты заранее понятны:
  - `22/tcp`
  - `80/tcp`
  - `443/tcp`, если `xmpp_stack_enable_nginx: true`
  - `5222/tcp`
  - `5269/tcp`, если включена federation
  - `5443/tcp`, если `xmpp_stack_enable_nginx: false`
  - `3478/tcp,udp`
  - `5349/tcp`
  - `49152-49200/udp`
- Внутренние порты не должны ожидаться снаружи:
  - `5443/tcp`, если `xmpp_stack_enable_nginx: true`
  - порты PostgreSQL

## Сертификаты

- `xmpp_stack_certbot_email` валиден.
- `xmpp_stack_certbot_domains` включает как минимум основной домен и `upload`.
- `conference` тоже включён, если нужен расширенный профиль и лучшая совместимость.
- Порт `80/tcp` доступен для первого ACME bootstrap.

## TURN И Медиа

- `xmpp_stack_turn_static_auth_secret` задан.
- `xmpp_stack_turn_realm` совпадает с нужным XMPP-доменом, если нет причины переопределять его.
- `xmpp_stack_public_ip` не является приватным или внутренним NAT-адресом.
- Диапазон relay-портов допустим для сервера и provider firewall.

## Control Host

- Установлен `ansible-core`.
- Локально работает `ansible-playbook`.
- Установлен `community.general`, если включено DNS automation.
- `ansible-lint` и, при желании, `pre-commit` доступны для локальной проверки.

## Сухая Проверка

- Выполните `ansible-playbook playbooks/bootstrap.yml --syntax-check`.
- Выполните `ansible-lint`.
- Ещё раз проверьте итоговые значения inventory перед первым bootstrap.
