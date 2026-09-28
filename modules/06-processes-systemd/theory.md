# Теория — процессы и systemd

## Процессы

```bash
ps aux | head
ps -ef --forest | head -40
top        # или htop
pgrep -a ssh
kill -TERM <pid>     # сначала мягко
kill -KILL <pid>     # крайняя мера
```

## systemd — менеджер служб и кучи всего

```bash
systemctl status ssh
systemctl list-units --type=service --state=running
systemctl is-enabled ssh
sudo systemctl restart ssh
sudo systemctl enable --now some.service
```

Журнал:

```bash
journalctl -u ssh -n 50 --no-pager
journalctl -b -p err --no-pager
```

## Unit-файл (идея)

```ini
[Unit]
Description=Lab hello service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/lab-hello.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

## Timers вместо классического cron (предпочтительно в systemd-мире)

`service` + `timer` дают календарь запусков и единый journal.

Классический cron всё ещё встречается:

```bash
crontab -e
sudo systemctl status cron    # Debian
sudo systemctl status crond   # RHEL
```
