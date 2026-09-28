# Лаборатория 10 — nginx + TLS + NFS

## Задача

HTTPS-сайт статуса с `cli`, NFS в lab-сеть, негативные тесты и мини-инцидент HTTP≠NFS.

## Подготовка

- `srv` + `cli`, firewall из 07, группа `opslab` из 03
- Снимок `before-services`

## Упражнения

### 1. Осмотр

```bash
ss -tulpn; ip -br a
systemctl is-active nginx 2>/dev/null || true
```

Запишите IP `srv` в lab-сети и CIDR для `/etc/exports`.

### 2. Изменение — nginx HTTP-контент

1. Установите nginx.
2. `/var/www/lab/index.html` — hostname, дата, ОС.
3. Server block на этот root; `nginx -t` + reload.
4. Локально и с `cli`: `curl -s http://…/`.

### 3. Изменение — TLS (обязательно)

1. Каталог `/etc/nginx/ssl/` (права на key: root, `600`).
2. Самоподписанный crt/key (CN=`srv.lab.local` или IP).
3. `listen 443 ssl` + протоколы ≥ TLSv1.2.
4. Либо `return 301 https://$host$request_uri` с `:80`, либо dual-listen с приоритетом HTTPS — зафиксируйте выбор в заметках (потом уйдёт в `HANDOFF.md`).
5. Firewall: `443/tcp` для lab (и/или нужной зоны).

Проверка с `cli`:

```bash
curl -vk https://<IP-srv>/ 2>&1 | tee ~/lab-notes/10-tls-curl.txt
openssl s_client -connect <IP-srv>:443 -servername srv.lab.local </dev/null 2>/dev/null | openssl x509 -noout -subject -dates
```

Если не вышло — сначала `nginx -t` и `ss -tln | grep 443`, потом firewall.

### 4. Изменение — 404 и негатив «сервис down»

1. Кастомная `404.html` + `error_page`.
2. Остановите nginx: с `cli` отличие «порт закрыт / connection refused» vs TLS handshake fail — в заметках.
3. Снова enable nginx.

### 5. Изменение — NFS

1. Share на `/srv/data/nfs/share` или `/srv/nfs/share`, `root:opslab` `2770`.
2. Экспорт **только** lab CIDR.
3. Клиент: mount, create file, видно на сервере.

### 6. Проверка — NFS негативы

Неверный путь экспорта; демонстрация `root_squash`; отказ записи вне `opslab` — сохраните выводы в `10-checks.txt`.

### 7. Проверка — мини-инцидент

HTTP(S) жив, NFS down (сервис или правило) → диагностика в `10-incident.md` → восстановление.

### 8. Автоматизация

`/usr/local/bin/lab-services-status.sh`: active nginx; слушает `:443` (и `:80` если нужно); `exportfs -v`; локальный `curl -k https://127.0.0.1/`. Exit 0 только если HTTPS OK.

### 9. Запись

Схема доступа; где лежат crt/key; как клиент доверяет (или почему `-k` только в lab); отсылка к модулям 13–15.

## Сдача

- [ ] HTTPS с `cli` отдаёт вашу страницу (сохранённый curl/openssl)
- [ ] TLS ≥ 1.2; key не world-readable
- [ ] HTTP либо редирект, либо явно вторичен и задокументирован
- [ ] Кастомная 404; отличие «сервис down» зафиксировано
- [ ] NFS в lab CIDR; негативы описаны
- [ ] Мини-инцидент закрыт; скрипт статуса зелёный по HTTPS

## Советы

| Тема | Debian/Ubuntu | Rocky/Alma |
|------|---------------|------------|
| Пакет nginx | `nginx` | `nginx` |
| NFS | `nfs-kernel-server` | `nfs-utils` |
| Открыть 443 | ufw/nft | `firewall-cmd --add-service=https` |

Если TLS fail из-за MAC — чините контекст/профиль (модуль 13), не `setenforce 0` навсегда.

## Не сносите

Сервисы и сертификаты оставьте для 12/15.
