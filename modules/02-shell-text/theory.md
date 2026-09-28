# Теория — оболочка (mid)

## Операторский минимум

```bash
echo $?                 # код возврата
cmd >out 2>err          # разделение потоков
cmd | other             # конвейер
set -euo pipefail       # каркас скриптов (с модуля 11 — must)
```

## Инструменты triage

```bash
grep -RInE 'error|fail|denied' /var/log 2>/dev/null | head
journalctl -p err..alert -n 100 --no-pager
awk -F: '$7 ~ /nologin|false/ {print $1}' /etc/passwd
cut -d: -f1 /etc/passwd | sort
find /var/log -type f -size +10M 2>/dev/null
diff -u file.bak file
```

## Безопасность

Перед разрушающим действием — та же маска через `ls`/`find`. Конфиги править через `.bak.$(date +%F%H%M)`.
