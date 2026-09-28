# Теория — журналы и мониторинг

## Где живут логи

- `journalctl` — основной интерфейс на systemd-системах
- `/var/log/` — текстовые логи приложений, иногда rsyslog

```bash
journalctl -b                          # текущая загрузка
journalctl --since "1 hour ago"
journalctl -u ssh -f                   # follow
journalctl -p err..alert -n 100
```

## Диск и retention

Журналы могут съесть диск:

```bash
journalctl --disk-usage
sudo journalctl --vacuum-size=200M
```

## Минимальный мониторинг без «сложной платформы»

Для старта админу достаточно:

1. доступность хоста (ping/ssh);
2. место на дисках;
3. failed units;
4. время ответа ключевого HTTP.

Позже это станет Prometheus/Zabbix — но привычка смотреть факты начинается здесь.
