# Лаборатория 10 — веб и NFS

## Цель

Опубликовать учебный сайт и NFS-шару для lab-сети, согласовать firewall и отработать отказ доступа / неверный экспорт.

## Окружение

- `srv` + `cli` (настоятельно рекомендуется)
- Firewall из модуля 07
- Группа `opslab` из модуля 03 (создайте, если нет)
- Снимок `before-services`

## Задания

### 1. Inspect

```bash
ss -tulpn
systemctl is-active nginx 2>/dev/null || true
df -h /srv /srv/data 2>/dev/null || df -h /srv
ip -br a
```

Запишите IP `srv` в lab-сети и подсеть для NFS-экспорта.

### 2. Change — nginx

1. Установите nginx.
2. Контент в `/var/www/lab/index.html`: hostname, дата, краткий статус (можно генерировать скриптом).
3. Server block на этот каталог (или аккуратная замена default).
4. `sudo nginx -t` + reload.
5. Локально: `curl -s http://127.0.0.1/` и с `cli`: `curl -s http://<IP-srv>/`.

| Семья | Установка |
|-------|-----------|
| Debian/Ubuntu | `sudo apt install nginx` |
| Rocky/Alma | `sudo dnf install nginx` + часто `sudo systemctl enable --now nginx` |

### 3. Change — кастомная ошибка и негативный тест HTTP

1. Создайте простую страницу 404 `/var/www/lab/404.html`.
2. Настройте `error_page 404 /404.html` в server block.
3. Запросите несуществующий URL с `cli` — убедитесь, что видите вашу 404.
4. Остановите nginx, с `cli` повторите `curl`/`nc` — зафиксируйте отличие «firewall ок, сервиса нет».

Снова запустите nginx.

### 4. Change — NFS

1. Каталог `/srv/data/nfs/share` (предпочтительно на LVM) или `/srv/nfs/share`.
2. Права: группа `opslab`, запись для группы (например `2770`).
3. Экспорт **только** в подсеть лаборатории (host-only), не `*`.
4. Включите nfs-server, `exportfs -rav`, `exportfs -v`.
5. На `cli` смонтируйте, создайте файл, увидьте на сервере.

| Семья | Пакет | Сервис |
|-------|-------|--------|
| Debian/Ubuntu | `nfs-kernel-server` | `nfs-server` |
| Rocky/Alma | `nfs-utils` | `nfs-server` |

Пример идеи `/etc/exports` (подставьте свою сеть):

```text
/srv/data/nfs/share  10.0.10.0/24(rw,sync,no_subtree_check)
```

### 5. Verify — NFS негативные тесты

1. С `cli` попробуйте смонтировать с **неверным** путём экспорта — зафиксируйте ошибку.
2. Создайте пользователя/файл так, чтобы продемонстрировать влияние `root_squash` (root на клиенте ≠ root на сервере) — 5–7 предложений в заметках.
3. Пользователь **вне** `opslab` не должен писать в share (если так задуманы права) — покажите отказ.

### 6. Firewall

Разрешите только нужное для lab-подсети:

- `80/tcp` (HTTP)
- NFS-связанные сервисы (`nfs`, `rpc-bind`, `mountd` на firewalld; на ufw — соответствующие порты/профиль)

**Не** открывайте NFS «в мир» / в NAT-интерфейс без необходимости.

### 7. Verify + мини-инцидент «сайт открыт, шара нет»

1. Временно уберите правило NFS / остановите `nfs-server`.
2. HTTP с `cli` работает, mount NFS — нет.
3. Верните сервис и правила.
4. Запишите шаги диагностики в `~/lab-notes/10-incident.md`.

### 8. Automate — статус сервисов

`/usr/local/bin/lab-services-status.sh` выводит active/inactive для `nginx`, `nfs-server`, слушает ли `:80`, есть ли экспорт (`exportfs -v`). Код 0 только если HTTP OK (NFS — warning в тексте, если down).

### 9. Document

`~/lab-notes/10-evidence.txt`: curl с cli, `exportfs -v`, `findmnt` на клиенте, firewall list.  
`~/lab-notes/10.md` — схема «кто к чему имеет доступ».

SELinux (Rocky/Alma): если отказ — смотрите `ausearch`/`journalctl`, учебные boolean/контексты; не отключайте SELinux «навсегда» без записи в заметках.

## Критерии приёмки

- [ ] HTTP с клиента отдаёт вашу страницу
- [ ] Кастомная 404 работает; отличие «нет сервиса» зафиксировано
- [ ] NFS смонтирован на клиенте, файл синхронизируется
- [ ] Негативные тесты NFS (неверный путь / права / root_squash) описаны
- [ ] Экспорт ограничен подсетью лаборатории
- [ ] Firewall согласован; мини-инцидент HTTP≠NFS закрыт
- [ ] Есть evidence, схема доступа, скрипт статуса

## Подсказки

- После правок exports: `exportfs -rav`.
- Проверяйте с `cli`, не только с localhost.
- На firewalld зона должна соответствовать интерфейсу lab-сети.

## Очистка

Сервисы оставьте — пригодятся в капстоуне.
