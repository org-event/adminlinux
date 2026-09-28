# Лаборатория 06 — процессы и systemd

## Цель

Разобрать процессы, оформить скрипт как systemd service + timer, отработать сбои unit и отличить `mask` / `disable` / `kill`.

## Окружение

- ВМ `srv`
- Снимок перед правкой unit-файлов желателен

## Задания

### 1. Осмотр — дерево процессов и failed units

```bash
ps -ef --forest | head -40
ps -o pid,ppid,user,stat,etime,cmd -p $$
systemctl list-units --type=service --state=failed
systemctl list-timers --all | head
journalctl -b -p err -n 30 --no-pager
```

1. Найдите PID вашей shell и PPID; запишите в `~/lab-notes/06-ps.txt`.
2. Найдите процесс с наибольшим RSS (`ps aux --sort=-rss | head`) — одной строкой в заметках.

### 2. Изменение — скрипт heartbeat

Сделайте `/usr/local/bin/lab-heartbeat.sh`:

```bash
#!/bin/bash
set -euo pipefail
LOG=/var/log/lab-heartbeat.log
umask 022
echo "$(date -Is) heartbeat host=$(hostname) load=$(cut -d' ' -f1-3 /proc/loadavg) pid=$$" \
  >> "$LOG"
```

Права: `755`, владелец `root:root`. Создайте лог-файл с корректными правами.

### 3. Изменение — service и timer

Сделайте:

- `/etc/systemd/system/lab-heartbeat.service` (`Type=oneshot`, `ExecStart=/usr/local/bin/lab-heartbeat.sh`)
- `/etc/systemd/system/lab-heartbeat.timer` (каждые **2 минуты**, `Persistent=true`)

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now lab-heartbeat.timer
systemctl list-timers | grep lab-heartbeat
systemctl cat lab-heartbeat.service lab-heartbeat.timer
```

### 4. Проверка — журнал и минимум два срабатывания

```bash
systemctl status lab-heartbeat.timer --no-pager
sudo journalctl -u lab-heartbeat.service -n 30 --no-pager
tail -5 /var/log/lab-heartbeat.log
```

Дождитесь **двух** записей в логе с разным временем. Если не вышло — смотрите `systemctl status lab-heartbeat.timer` и `journalctl -u lab-heartbeat.service`.

### 5. Изменение — сигналы и процессы

1. Запустите в одном терминале: `sleep 300 &` — запомните PID.
2. Отправьте `SIGSTOP`, проверьте `STAT` в `ps`, затем `SIGCONT`.
3. Завершите через `SIGTERM`, убедитесь что процесса нет.
4. Кратко в заметках: чем `kill` / `kill -9` отличаются для админа.

### 6. Проверка — мини-инцидент failed unit

1. Сломайте `ExecStart` в service (несуществующий путь), `daemon-reload`, запустите service вручную.
2. Сохраните вывод `systemctl status` + `journalctl -u` в `~/lab-notes/06-incident.txt`.
3. Верните правильный путь, `daemon-reload`, `start` — unit снова успешен.
4. При необходимости покажите `systemctl reset-failed`.

### 7. Проверка — disable vs mask

На **учебном** timer:

```bash
sudo systemctl stop lab-heartbeat.timer
sudo systemctl disable lab-heartbeat.timer
# затем верните enable --now

# отдельно продемонстрируйте mask (и сразу unmask!):
sudo systemctl mask lab-heartbeat.timer
systemctl status lab-heartbeat.timer --no-pager || true
sudo systemctl unmask lab-heartbeat.timer
sudo systemctl enable --now lab-heartbeat.timer
```

В `~/lab-notes/06.md` одной таблицей: `stop` / `disable` / `mask`.

### 8. Автоматизация — oneshot по требованию

Добавьте возможность ручного прогона без ожидания timer:

```bash
sudo systemctl start lab-heartbeat.service
```

Убедитесь, что в логе появилась новая строка.

### 9. Запись

В `~/lab-notes/06.md`: разница `service` и `timer`; куда смотреть при сбое (`status`, `journalctl -u`, `systemctl --failed`).

## Критерии приёмки

- [ ] Скрипт исполняемый и пишет в `/var/log/lab-heartbeat.log`
- [ ] Timer активен, видно в `list-timers`, ≥2 срабатывания
- [ ] Отработаны сигналы STOP/CONT/TERM
- [ ] Мини-инцидент failed unit: сломали → нашли в journal → починили
- [ ] Понятна разница disable vs mask (с unmask)
- [ ] Есть заметки и сохранённый вывод по инциденту

## Подсказки

- После правок unit всегда `daemon-reload`.
- `Type=oneshot` + timer — типичный паттерн для замены cron.
- Не маскируйте системные службы (`ssh`/`sshd`, `network`) «для эксперимента».

## Очистка (по желанию)

```bash
sudo systemctl disable --now lab-heartbeat.timer
# unit-файлы можно оставить до капстоуна
```
