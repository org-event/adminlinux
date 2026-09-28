# Лаборатория 02 — оболочка и текст

## Цель

Находить информацию в файлах и журналах командами, а не «на глаз», и фиксировать доказательства в файлах.

## Окружение

- ВМ `srv` после модуля 01
- Права `sudo` для чтения части логов

## Задания

### 1. Inspect — навигация

```bash
cd ~
pwd
ls -la /
ls -ld /etc /var /var/log
```

Создайте `~/lab-notes/02-workspace` и три файла `a.txt`, `b.txt`, `c.txt` с разным содержимым (несколько строк слов).

### 2. Change — конвейеры

1. Объедините файлы в `all.txt`.
2. Посчитайте строки/слова/байты (`wc`).
3. Топ частот слов (`tr`/`sort`/`uniq`).

```bash
cd ~/lab-notes/02-workspace
cat a.txt b.txt c.txt > all.txt
wc all.txt
tr -s '[:space:]' '\n' < all.txt | tr 'A-Z' 'a-z' | sort | uniq -c | sort -nr | head
```

### 3. Verify — разбор /etc/passwd

```bash
awk -F: '{print $1 " -> " $7}' /etc/passwd | tee ~/lab-notes/02-shells.txt
awk -F: '$7 ~ /(nologin|false)$/ {print $1, $7}' /etc/passwd | tee ~/lab-notes/02-nologin.txt
```

### 4. Change — find и безопасный поиск

1. Найдите в `/etc` файлы `*.conf`, изменённые за последние 7 дней (`find`), сохраните список (ошибок permission — в stderr отдельно или `2>/dev/null` после того, как поняли зачем).
2. Найдите файлы > 10M в `/var/log` (если есть) — только список путей.

```bash
sudo find /etc -name '*.conf' -mtime -7 2>/dev/null | head -50 | tee ~/lab-notes/02-find-conf.txt
```

### 5. Change — diff, cut, простой sed

1. Скопируйте `all.txt` в `all-edit.txt`, измените одну строку.
2. Покажите `diff -u all.txt all-edit.txt`.
3. Из `/etc/passwd` выведите только username через `cut -d: -f1 | head`.
4. Одним `sed` замените слово в копии рабочего файла (не в системных!).

### 6. Verify — журналы и код возврата

```bash
sudo journalctl -p err..alert -n 50 --no-pager | tee ~/lab-notes/02-errors.txt
# или: sudo grep -iE 'error|fail' /var/log/syslog 2>/dev/null | tail -10
```

Практика `$?`:

```bash
true; echo "true => $?"
false; echo "false => $?"
ls /no/such/path; echo "ls missing => $?"
```

Сохраните выводы в `~/lab-notes/02-exitcodes.txt`.

### 7. Automate — мини-отчёт одной командой

Скрипт `~/lab-notes/bin/lab02-report.sh`: собирает `wc all.txt`, число nologin-пользователей, хвост error-выборки. Запустите и приложите вывод.

### 8. Document

В `~/lab-notes/02.md`: разница `>` и `>>`; зачем `$?`; чем опасен `rm` с неверным путём после `cd`.

## Критерии приёмки

- [ ] Есть `02-workspace` с исходниками и `all.txt`
- [ ] Есть `02-shells.txt` и выборка nologin
- [ ] Есть find/diff/exitcodes evidence
- [ ] Есть выборка ошибок журнала
- [ ] Есть report-скрипт и заметки Document
- [ ] Ничего системного не удалено

## Подсказки

- Нет `/var/log/syslog` — смотрите `journalctl` или `/var/log/messages`.
- `2>/dev/null` скрывает ошибки — для отладки сначала уберите.

## Очистка

Можно удалить только `~/lab-notes/02-workspace`, заметки оставьте.
