# Теория — веб, TLS, NFS

## Nginx

```bash
# Debian/Ubuntu: apt install nginx
# Rocky/Alma:    dnf install nginx
sudo nginx -t && sudo systemctl reload nginx
```

Конфиги: `/etc/nginx/`. Корень сайта (document root) учебной площадки: `/var/www/lab`. Всегда `nginx -t` перед reload.

## TLS (обязательно для mid+)

В лаборатории достаточно **самоподписанного** сертификата (или внутреннего CA). Цель — привычка: HTTPS, редирект/раздельные listen, проверка с клиента, открытый `443/tcp` в firewall.

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/lab.key \
  -out /etc/nginx/ssl/lab.crt \
  -subj "/CN=srv.lab.local"
```

В server block: `listen 443 ssl;`, пути к crt/key, `ssl_protocols TLSv1.2 TLSv1.3;`.  
С клиента: `curl -vk https://<IP>/` (или установка учебного CA в trust store `cli`).

HTTP (80) — либо редирект на HTTPS, либо явная статус-страница с ссылкой. «Только plaintext навсегда» для mid+ **не принимается**.

## NFS

Экспорт **только** в host-only/lab подсеть. `exportfs -rav`, проверка с `cli`. Учитывайте `root_squash`.

| Семья | Пакет | Сервис |
|-------|-------|--------|
| Debian/Ubuntu | `nfs-kernel-server` | `nfs-server` |
| Rocky/Alma | `nfs-utils` | `nfs-server` |

## Безопасность

- Слушать нужный интерфейс; firewall: `443/tcp` (+ `80` если нужен), NFS только lab.
- SELinux/AppArmor могут блокировать TLS-пути и NFS — см. модуль 13. Здесь: не отключайте MAC «навсегда».
