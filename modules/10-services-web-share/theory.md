# Теория — веб и обмен файлами

## Nginx как учебный веб-сервер

```bash
# Debian
sudo apt install -y nginx
# RHEL
sudo dnf install -y nginx

sudo systemctl enable --now nginx
curl -I http://127.0.0.1/
```

Конфиги: `/etc/nginx/`. После правок:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## NFS — сетевая ФС для «внутренней» сети

Сервер экспортирует каталог, клиент монтирует.

```bash
# идеи файлов
/etc/exports
sudo exportfs -a
sudo systemctl enable --now nfs-server   # имя может отличаться
```

На клиенте:

```bash
sudo mount -t nfs srv:/srv/nfs/share /mnt
```

## Безопасность служб

- Слушайте на нужном интерфейсе.
- Firewall: только необходимые порты.
- Для NFS не публикуйте экспорт в интернет — только lab-сеть.
- Отдельный пользователь/права на document root и share.
