# Лаборатория 02 — triage текстом

## Цель

Соберите пакет доказательств по журналам и `/etc` командами; оформите короткий отчёт (однострочник или скрипт).

## Окружение

- `srv` после модуля 01
- Для журналов нужен `sudo`

## Задания

### 1. Осмотр — карта логов

```bash
ls -lah /var/log | tee ~/lab-notes/02-varlog.txt
sudo journalctl --disk-usage
sudo journalctl -p err..alert -n 80 --no-pager | tee ~/lab-notes/02-errors.txt
```

### 2. Изменение — разбор учёток и шеллов

```bash
awk -F: '{print $1 "\t" $7}' /etc/passwd | tee ~/lab-notes/02-shells.txt
awk -F: '$7 ~ /(nologin|false)$/ {print $1, $7}' /etc/passwd | tee ~/lab-notes/02-nologin.txt
```

Сколько nologin? Запишите число в `02.md`.

### 3. Проверка — find + размер

1. `*.conf` в `/etc` с mtime −7 дней → `02-find-conf.txt` (stderr обработайте осознанно).
2. Файлы в `/var/log` больше 5M → список путей.

### 4. Изменение — diff конфига

1. `sudo cp /etc/hosts /etc/hosts.bak.lab02`
2. Добавьте **учебную** строку-комментарий в `/etc/hosts`.
3. `diff -u` → `~/lab-notes/02-hosts.diff`
4. Откатите из `.bak`.

### 5. Проверка — exit codes

Сохраните в `02-exitcodes.txt`:

```bash
true; echo "true:$?"
false; echo "false:$?"
grep -q 'NoSuchPatternXYZ' /etc/passwd; echo "grep_no_match:$?"
```

В `02.md` напишите, как используете `$?` в скрипте бэкапа или health-check.

### 6. Автоматизация — report

`~/lab-notes/bin/lab02-report.sh`: число nologin, хвост error-журнала (≤20 строк), `journalctl --disk-usage`. Exit 0.

### 7. Запись

`02.md`: `>` vs `>>`; когда `2>/dev/null` вреден; один пример ложного вывода из-за неправильного pipe.

## Критерии приёмки

- [ ] Есть `02-errors.txt`, shells/nologin, вывод find в `~/lab-notes/`
- [ ] Есть `02-hosts.diff` и откат hosts
- [ ] Exit codes зафиксированы
- [ ] Report-скрипт работает
- [ ] Системные файлы не удалены

## Подсказки

| Семья | Текстовый syslog (если есть) |
|-------|------------------------------|
| Debian/Ubuntu | `/var/log/syslog` или только journal |
| Rocky/Alma | `/var/log/messages` или journal |

Если не вышло с journal — проверьте права через `sudo` и что journald вообще пишет.

## Очистка

Удалите только временные копии в home; `.bak` hosts уберите после отката.
