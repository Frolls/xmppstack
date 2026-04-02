# Руководство По Развёртыванию

## Обзор

Этот репозиторий разворачивает приватный XMPP-стек через Ansible. Целевая
runtime-схема основана на Podman Quadlet и systemd на Linux-хосте.

Управляемые сервисы:
- `ejabberd` для XMPP, синхронизации между устройствами, комнат, upload, push и WebSocket
- `postgres` для архива сообщений и серверных данных
- `coturn` для STUN/TURN и более надёжных звонков за NAT
- `nginx` для HTTP/HTTPS-входа на `80/443`
- `certbot` для выпуска и продления сертификатов
- `firewall` для host-level фильтрации через nftables
- опциональная автоматизация `dns` для зон у провайдера

Репозиторий оформлен в стиле Ansible collection, но используется как единый
локальный проект.

## Структура Репозитория

- `galaxy.yml`
  Метаданные collection.
- `meta/runtime.yml`
  Требования к версии Ansible.
- `roles/common/`
  Общая валидация, системные пакеты, каталоги, `.env`, сеть Podman и вспомогательные скрипты.
- `roles/postgres/`
  Quadlet и lifecycle PostgreSQL.
- `roles/dns/`
  Опциональная роль управления DNS. Сейчас реализованы Cloudflare и Reg.ru.
- `roles/firewall/`
  Роль host-level firewall на базе nftables, согласованная с портами стека.
- `roles/ejabberd/`
  Конфиг ejabberd, локальная сборка образа, Quadlet и lifecycle сервиса.
- `roles/coturn/`
  Конфиг coturn, локальная сборка образа, Quadlet и lifecycle сервиса.
- `roles/nginx/`
  Конфиг nginx, Quadlet и lifecycle сервиса.
- `roles/certbot/`
  Bootstrap сертификата, скрипты продления и renewal timer.
- `playbooks/`
  Точки входа для bootstrap и обычного deploy.
- `inventories/production/`
  Пример production inventory.
- `docs/`
  Эксплуатационная и вспомогательная документация.
- `ci/`
  Минимальные заготовки под CI.

## Порядок Выполнения

Playbooks применяют роли в таком порядке:
1. `common`
2. `firewall`
3. `dns`
4. `postgres`
5. `ejabberd`
6. `coturn`
7. `nginx`
8. `certbot`

Этот порядок важен:
- общие каталоги и Podman network должны существовать до сервисных ролей
- firewall применяется до старта сервисов, чтобы открытые порты были явно описаны
- если включено автоматическое управление DNS, записи должны появиться до первого bootstrap сертификата
- `postgres` должен быть доступен до старта ejabberd
- `ejabberd`, `coturn` и `nginx` сначала раскладывают свои unit-файлы и конфиги
- `certbot` выпускает первый сертификат и затем перезапускает TLS-зависимые сервисы

## Требования

Рекомендуемый целевой сервер:
- `2 vCPU`
- `4 GB RAM`
- `40+ GB SSD/NVMe`
- публичный `IPv4`
- `IPv6` по возможности

Рекомендуемый управляющий хост:
- `ansible-core >= 2.15`
- SSH-доступ к серверу
- возможность использовать `become`
- опционально: `pre-commit` для локальной проверки перед push

Ожидания от целевого сервера:
- есть systemd
- пакетный менеджер может установить `podman`
- firewall позволяет открыть нужные порты

## Runtime-Структура На Сервере

Роли создают и используют:
- `/opt/xmpp-infra`
  Рендеренные runtime-конфиги, скрипты и контексты сборки контейнеров.
- `/srv/xmpp/postgres`
  Данные PostgreSQL.
- `/srv/xmpp/ejabberd/upload`
  Загруженные файлы.
- `/srv/xmpp/letsencrypt`
  Состояние сертификатов.
- `/srv/xmpp/backups`
  Каталог для бэкапов.

## Структура Inventory

Используйте production inventory в таком виде:

- `inventories/production/hosts.yml`
- `inventories/production/group_vars/xmpp_hosts/main.yml`
- `inventories/production/group_vars/xmpp_hosts/vault.yml`

Секреты должны лежать в `vault.yml` и быть зашифрованы через `ansible-vault`.

### Несекретные переменные

Обычно в `main.yml` задаются:
- `xmpp_stack_domain`
- `xmpp_stack_upload_host`
- `xmpp_stack_conference_host`
- `xmpp_stack_public_ip`
- `xmpp_stack_public_ipv6`
- `xmpp_stack_admin_jid`
- `xmpp_stack_certbot_email`
- `xmpp_stack_certbot_domains`
- `xmpp_stack_manage_dns`
- `xmpp_stack_dns_provider`
- `xmpp_stack_dns_zone`

### Секретные переменные

Обычно в `vault.yml` задаются:
- `xmpp_stack_secrets.postgres_password`
- `xmpp_stack_secrets.turn_static_auth_secret`
- `xmpp_stack_dns_secrets.cloudflare_api_token`
- `xmpp_stack_dns_secrets.regru_username`
- `xmpp_stack_dns_secrets.regru_password`

Примеры inventory под конкретных провайдеров:
- `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.main.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.vault.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/regru.main.yml.example`
- `inventories/production/group_vars/xmpp_hosts/examples/regru.vault.yml.example`

## Важные Переменные

Основные переменные:
- `xmpp_stack_domain`
  Основной XMPP-домен.
- `xmpp_stack_upload_host`
  Хост для HTTP upload.
- `xmpp_stack_conference_host`
  Домен для MUC-комнат.
- `xmpp_stack_public_ip`
  Публичный IP для TURN.
- `xmpp_stack_public_ipv6`
  Опциональный IPv6 для генерации `AAAA`-записей.
- `xmpp_stack_admin_jid`
  Административный JID.
- `xmpp_stack_certbot_email`
  Email для Let's Encrypt.
- `xmpp_stack_certbot_domains`
  Список доменов для выпуска сертификата.
  В расширенном профиле сюда обычно входят основной XMPP-домен, `upload` и
  `conference`.

Секреты:
- `xmpp_stack_secrets.postgres_password`
- `xmpp_stack_secrets.turn_static_auth_secret`

Операционные переключатели:
- `xmpp_stack_issue_certificates`
  Разрешает ли роль `certbot` выпускать первый сертификат.
- `xmpp_stack_build_images`
  Нужно ли ролям `ejabberd` и `coturn` собирать локальные образы.
- `xmpp_stack_manage_dns`
  Нужно ли опциональной роли `dns` управлять DNS-записями.
- `xmpp_stack_log_level`
  Уровень логирования `ejabberd`.
- `xmpp_stack_enable_federation`
  Включает listener на `5269/tcp` и s2s-модули ejabberd.
- `xmpp_stack_enable_invites`
  Включает `mod_invites` для инвайтов без открытия публичной регистрации.

Переменные тюнинга ejabberd:
- `xmpp_stack_invites_landing_page`
- `xmpp_stack_invites_max`
- `xmpp_stack_invites_token_expire_seconds`
- `xmpp_stack_c2s_shaper_rate`
- `xmpp_stack_s2s_shaper_rate`
- `xmpp_stack_max_user_sessions`
- `xmpp_stack_max_user_offline_messages`

DNS-переменные:
- `xmpp_stack_dns_provider`
- `xmpp_stack_dns_zone`
- `xmpp_stack_dns_ttl`
- `xmpp_stack_dns_proxied`
- `xmpp_stack_dns_manage_srv`
- `xmpp_stack_dns_manage_txt`
- `xmpp_stack_websocket_url`
- `xmpp_stack_dns_records`
- `xmpp_stack_dns_srv_records`
- `xmpp_stack_dns_txt_records`

Пути:
- `xmpp_stack_app_root`
- `xmpp_stack_data_root`
- `xmpp_stack_postgres_data_dir`
- `xmpp_stack_upload_dir`
- `xmpp_stack_letsencrypt_dir`
- `xmpp_stack_backups_dir`

## Playbooks

- `playbooks/bootstrap.yml`
  Путь для первого запуска, включая выпуск первого сертификата.
- `playbooks/deploy.yml`
  Путь для обычных обновлений после первичного развёртывания.
- `playbooks/site.yml`
  Универсальный all-in-one запуск.

### Рекомендуемое Использование

Первый запуск:

```sh
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass
```

Обычные обновления:

```sh
ansible-playbook playbooks/deploy.yml --ask-vault-pass
```

## Теги

Общие теги:
- `validate`
- `packages`
- `directories`
- `config`
- `images`
- `bootstrap`
- `certificates`
- `services`

Теги ролей:
- `common`
- `firewall`
- `dns`
- `postgres`
- `ejabberd`
- `coturn`
- `nginx`
- `certbot`

Примеры точечных запусков:

```sh
ansible-playbook playbooks/deploy.yml --ask-vault-pass --tags config,ejabberd
ansible-playbook playbooks/bootstrap.yml --ask-vault-pass --tags certificates,certbot
```

## Логика Развёртывания

Playbooks выполняют такие высокоуровневые этапы:

1. Проверяют обязательные переменные.
2. Устанавливают системные пакеты.
3. Создают рабочие каталоги.
4. Рендерят общие runtime-артефакты и Podman network.
5. Применяют nftables firewall для хоста.
6. Опционально управляют DNS-записями через выбранного провайдера.
7. Рендерят Quadlet-файлы и сервисные конфиги.
8. При необходимости собирают локальные образы `ejabberd` и `coturn`.
9. Запускают `postgres` и включают TLS-зависимые сервисы.
10. Если первого сертификата нет и это разрешено, выполняют bootstrap через `certbot`.
11. Включают timer для продления сертификатов.

Логика сборки образов завязана на hash stamp, поэтому пересборка происходит
только при изменении входных файлов.

TLS-зависимые сервисы включаются заранее, но стартуют сразу только если
сертификаты уже существуют. При самом первом запуске `certbot` выпускает
сертификаты и затем перезапускает `ejabberd`, `coturn` и `nginx`.

## DNS

См. также:
- [dns.md](./dns.md)
- [DNS_PROVIDERS.md](./DNS_PROVIDERS.md)
- [DNS_PROVIDERS.ru.md](./DNS_PROVIDERS.ru.md)

Минимально обычно нужны:
- `A/AAAA` для `xmpp_stack_domain`
- `A/AAAA` для `xmpp_stack_upload_host`
- `A/AAAA` для `xmpp_stack_conference_host`

Дополнительные SRV-записи улучшают совместимость:
- `_xmpp-client._tcp.<domain>`
- `_xmpp-server._tcp.<domain>`

Если включить роль `dns` с Cloudflare, эти записи можно создавать прямо из
Ansible. Поддержка сделана через `community.general.cloudflare_dns`, поэтому на
управляющем хосте должна быть установлена коллекция `community.general`.

## Порты

См. также:
- [ports.md](./ports.md)

Обычно открываются:
- `5222/tcp` для XMPP client-to-server
- `5269/tcp` для XMPP server-to-server
- `5443/tcp` для backend HTTP upload и WebSocket у ejabberd
- `3478/tcp,udp` для STUN/TURN
- `5349/tcp` для TURN over TLS
- `49152-49200/udp` для TURN relay
- `80/tcp` и `443/tcp` для `nginx` и `certbot`

## Сертификаты

Сертификаты хранятся в:
- `/srv/xmpp/letsencrypt`

Первичный выпуск делается через:
- `scripts/certbot-bootstrap.sh`, который рендерит роль `certbot`

Продление делается через:
- `certbot-renew.service`
- `certbot-renew.timer`

Во время bootstrap или renewal proxy-сервис может кратко останавливаться, чтобы
`certbot --standalone` занял порт `80`.

## Эксплуатация

См. также:
- [operations.md](./operations.md)
- [secrets.md](./secrets.md)

Рекомендуемые регулярные проверки:
- убедиться, что сервисы активны
- проверить, что бэкапы создаются
- проверить мобильный push
- проверить upload
- периодически делать реальный звонок из внешней сети

## Диагностика

Если playbook падает в начале:
- проверьте, что `vault.yml` существует и передан через `--ask-vault-pass`
- проверьте, что сервер доступен по SSH
- проверьте, что `become` работает

Если не выпускается сертификат:
- проверьте, что DNS уже указывает на сервер
- проверьте доступность `80/tcp` из интернета
- проверьте, что порт `80` не занят другим сервисом

Если не работают звонки:
- проверьте, что `xmpp_stack_public_ip` указан корректно
- проверьте, что открыты TURN-порты
