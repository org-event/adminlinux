# Теория — окружение (mid)

## Две семьи — одна работа

| | Debian/Ubuntu | Rocky/Alma |
|--|---------------|------------|
| Пакеты | `apt` / `dpkg` | `dnf` / `rpm` |
| Firewall (типично) | nftables / ufw | firewalld |
| MAC | AppArmor | SELinux (модуль 13) |
| Сеть | часто netplan / ifupdown | NetworkManager |

Админ переносит **модель** (конфиг в `/etc`, юниты systemd, журналы), а не заучивает один дистрибутив.

## FHS как карта инцидента

| Путь | Когда смотреть |
|------|----------------|
| `/etc` | «кто сломал конфиг» |
| `/var/log`, journal | «что случилось» |
| `/home`, `/srv` | данные людей/приложений |
| `/usr` | поставленное ПО |
| `/proc`, `/sys` | живое ядро (не бэкапить как файлы) |

## Root / sudo

Рабочий режим: обычный пользователь + `sudo -l`. Полный интерактивный root — исключение с записью в заметках.

## Базовый осмотр (запомните как ритуал)

```bash
cat /etc/os-release; uname -r; hostnamectl
ip -br a; free -h; df -hT; lsblk
systemctl is-system-running
```
