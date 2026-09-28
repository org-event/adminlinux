# Теория — сеть и firewall

## Базовая диагностика

```bash
ip -br a
ip r
ip neigh
resolvectl status 2>/dev/null || cat /etc/resolv.conf
ping -c 3 1.1.1.1
ping -c 3 example.com
curl -I https://example.com
ss -tulpn
```

Разделяйте проблемы:

1. L2/L3 (линк, маршрут, ping по IP)
2. DNS (имя не резолвится)
3. Прикладной порт (слушает ли служба, пускает ли firewall)

## Firewall: две школы

### firewalld (RHEL-семейство)

```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

### nftables / ufw (Debian-семейство)

```bash
# ufw (проще для старта на Ubuntu)
sudo ufw status verbose
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw enable

# nft
sudo nft list ruleset
```

## Правило безопасности

Перед включением firewall **убедитесь**, что SSH разрешён, иначе потеряете доступ к ВМ. Держите консоль гипервизора под рукой.
