# Теория — Mandatory Access Control

## Зачем MAC

DAC (chmod/sudo) не спасает, если скомпрометирован процесс. MAC ограничивает *что процесс имеет право делать* сверх UID.

| Семья | Подсистема | Типичный режим «вкл» |
|-------|------------|----------------------|
| Rocky/Alma | SELinux | Enforcing |
| Debian/Ubuntu | AppArmor | profiles enabled/enforce |

## SELinux — цикл фикса

```bash
getenforce
sudo ausearch -m avc -ts recent
sudo journalctl -t setroubleshoot -n 50   # если пакет есть
# типичные фиксы:
sudo setsebool -P httpd_can_network_connect 1
sudo restorecon -Rv /var/www/lab
sudo semanage fcontext -a -t httpd_sys_content_t '/var/www/lab(/.*)?'
```

`setenforce 0` — **только** краткий диагностический шаг с записью времени. Финал лаборатории — снова **Enforcing** и рабочий сервис. Не отключайте SELinux «навсегда».

## AppArmor — цикл фикса

```bash
sudo aa-status
sudo journalctl -b | grep -i apparmor
# /var/log/syslog или audit: DENIED
sudo aa-complain /etc/apparmor.d/...   # временно для сбора
# правка профиля / aa-logprof — затем снова enforce
```

Полный `systemctl disable apparmor` как «решение» — **не принимается**.

## Связь с модулем 10

Нестандартные пути TLS (`/etc/nginx/ssl`), document root вне default, NFS — частые источники deny. Чините контекст/профиль. Не выкидывайте MAC.
