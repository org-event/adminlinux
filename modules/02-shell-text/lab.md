# Лаборатория 02 — оболочка и текст

## Цель

Научиться находить информацию в файлах и журналах и оформлять результат командами, а не «на глаз».

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

Создайте рабочий каталог `~/lab-notes/02-workspace` и три файла `a.txt`, `b.txt`, `c.txt` с разным содержимым.

### 2. Change — конвейеры

1. Объедините файлы в `all.txt`.
2. Посчитайте число строк, слов и байт (`wc`).
3. Сделайте список уникальных слов (примитивно через `tr`/`sort`/`uniq`) и сохраните топ-частот.

Пример каркаса:

```bash
cd ~/lab-notes/02-workspace
cat a.txt b.txt c.txt > all.txt
wc all.txt
tr -s '[:space:]' '\n' < all.txt | tr 'A-Z' 'a-z' | sort | uniq -c | sort -nr | head
```

### 3. Verify — разбор /etc/passwd

Выведите список пользователей и их login shell в формате `user -> shell`:

```bash
awk -F: '{print $1 " -> " $7}' /etc/passwd | tee ~/lab-notes/02-shells.txt
```

Отдельно найдите всех с shell `nologin` или `false`.

### 4. Журналы

Найдите в журнале (файл или `journalctl`) 10 последних строк с `error`/`fail` (регистр не важен) и сохраните в `~/lab-notes/02-errors.txt`.

```bash
# вариант systemd
sudo journalctl -p err..alert -n 50 --no-pager | tee ~/lab-notes/02-errors.txt

# или файлы
sudo grep -iE 'error|fail' /var/log/syslog 2>/dev/null | tail -10
```

### 5. Document

В `~/lab-notes/02.md` опишите одной фразой разницу между `>` и `>>`, и зачем нужен `$?`.

## Критерии приёмки

- [ ] Есть `02-workspace` с исходными файлами и `all.txt`
- [ ] Есть `02-shells.txt` с пользователями и shell
- [ ] Есть выборка ошибок журнала
- [ ] В заметках объяснены `>` / `>>` и `$?`
- [ ] Ничего системного не удалено случайно

## Подсказки

- Если `/var/log/syslog` нет — смотрите `journalctl` или `/var/log/messages`.
- `2>/dev/null` скрывает ошибки — для отладки сначала уберите его.

## Очистка

Можно удалить только `~/lab-notes/02-workspace`, заметки оставьте.
