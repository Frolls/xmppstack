# DNS-провайдеры

В этом проекте опциональная роль `dns` поддерживает два backend-провайдера:
- `cloudflare`
- `regru`

## Рекомендация

Если вы можете выбирать свободно, по умолчанию лучше использовать `cloudflare`.

`regru` стоит выбирать, если:
- ваша авторитативная DNS-зона уже размещена в Reg.ru
- вы не хотите переносить зону к другому провайдеру
- вам достаточно более узкой автоматизации под нужды этого стека

## Сравнение

| Провайдер | Статус в проекте | Сильные стороны | Ограничения | Рекомендация по умолчанию |
| --- | --- | --- | --- | --- |
| `cloudflare` | Полностью реализован через `community.general.cloudflare_dns` | Чистая интеграция с Ansible, проще идемпотентность, легче сопровождать в долгую | Нужна коллекция `community.general` на управляющем хосте, отдельное управление токеном/API-доступом | Да |
| `regru` | Реализован через прямые вызовы `REG.API 2` с помощью `uri` | Удобен, если зона уже живёт в Reg.ru и не хочется делать миграцию DNS | Реализация уже, чем у Cloudflare, и сильнее привязана к форме API | Только если DNS уже там |

## Практические замечания

### Cloudflare

- Лучший вариант, если нужен наиболее сопровождаемый Ansible-путь.
- Хорошо покрывает задачи этого стека:
  - apex `A`
  - `A` для upload host
  - `A` для conference host
  - опциональные `SRV`
- Для XMPP-записей обычно стоит держать `proxied: false`, если у вас нет очень конкретной причины сделать иначе.

### Reg.ru

- Лучший вариант, если DNS уже управляется в Reg.ru и вы не хотите переносить зону.
- Текущий backend намеренно сфокусирован на записях, нужных этому стеку:
  - `A`
  - `AAAA`
  - `CNAME`
  - базовые `SRV`
- Это практичный слой автоматизации, а не полный универсальный DNS-CRUD для всех возможных сценариев.

## Примеры Inventory

- Cloudflare:
  - `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.main.yml.example`
  - `inventories/production/group_vars/xmpp_hosts/examples/cloudflare.vault.yml.example`
- Reg.ru:
  - `inventories/production/group_vars/xmpp_hosts/examples/regru.main.yml.example`
  - `inventories/production/group_vars/xmpp_hosts/examples/regru.vault.yml.example`
