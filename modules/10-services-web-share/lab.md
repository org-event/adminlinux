# Лаборатория 10 — веб и NFS

## Цель

Опубликовать учебный сайт и NFS-шару для внутренней сети лаборатории.

## Окружение

- `srv` + `cli` (настоятельно рекомендуется)
- Firewall из модуля 07
- Снимок `before-services`

## Задания

### 1. Inspect

```bash
ss -tulpn
systemctl is-active nginx || true
df -h /srv
```

### 2. Change — nginx

1. Установите nginx.
2. Создайте site content в `/var/www/lab/` (index.html с hostname и датой).
3. Настройте server block на этот каталог (или замените default аккуратно).
4. `nginx -t` + reload.
5. Проверьте локально и с `cli`: `curl http://<IP-srv>/`.

### 3. Change — NFS

1. Каталог `/srv/nfs/share` (можно на томе `/srv/data/nfs/share`).
2. Права: группа `opslab` или отдельная, запись для учебной группы.
3. Экспорт только в подсеть лаборатории (например `10.0.10.0/24` или вашу host-only).
4. Включите nfs-server, `exportfs -v`.
5. На `cli` смонтируйте шару, создайте файл, увидьте его на сервере.

### 4. Firewall

Разрешите:

- `80/tcp` (HTTP)
- NFS-порты/сервисы для вашей lab-подсети (на firewalld часто `nfs`, `rpc-bind`, `mountd`)

Не открывайте NFS «в мир».

### 5. Verify + Document

Соберите `~/lab-notes/10-evidence.txt`: curl с cli, `exportfs -v`, `findmnt` на клиенте.  
В `~/lab-notes/10.md` — схема «кто к чему имеет доступ».

## Критерии приёмки

- [ ] HTTP с клиента отдаёт вашу страницу
- [ ] NFS смонтирован на клиенте, файл синхронизируется
- [ ] Экспорт ограничен подсетью лаборатории
- [ ] Firewall отражает нужные сервисы
- [ ] Есть evidence и схема доступа

## Подсказки

| Семья | Пакет NFS | Сервис |
|-------|-----------|--------|
| Debian | `nfs-kernel-server` | `nfs-server` |
| RHEL | `nfs-utils` | `nfs-server` |

SELinux на RHEL может мешать контекстам — если отказано в доступе, смотрите `ausearch`/`journalctl` и учебные boolean/контексты (не отключайте SELinux «навсегда» без записи в заметках).

## Очистка

Сервисы оставьте — пригодятся в капстоуне.
