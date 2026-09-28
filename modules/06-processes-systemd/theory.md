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

Сначала мягкий сигнал. `SIGKILL` — когда процесс уже не отвечает на уговоры.

## systemd — менеджер служб (и многого ещё)

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

При сбое службы смотрите `systemctl status` и сразу `journalctl -u`.

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

После любой правки unit — `daemon-reload`, иначе systemd читает старое.

## Timers вместо классического cron

В мире systemd пара `service` + `timer` удобнее: календарь запусков и один journal.

Классический cron всё ещё встречается:

```bash
crontab -e
sudo systemctl status cron    # Debian
sudo systemctl status crond   # RHEL
```
