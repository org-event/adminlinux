# Лаборатория 03 — least privilege

## Цель

Собрать модель `opslab` с ACL и ограниченным sudo; каждый отказ доступа — в evidence.

## Окружение

- `srv`, снимок перед sudoers/ACL

## Задания

### 1. Inspect

```bash
id; umask; sudo -l
getent passwd | tail -8
ls -ld /home /srv 2>/dev/null
```

### 2. Change — роли

- группа `opslab`
- `dev1`, `ops1` (home + bash) ∈ `opslab`
- `guest1` **вне** группы
- `/srv/opslab` → `root:opslab` `2770`

### 3. Change — файл + ACL

1. От `dev1`: `/srv/opslab/notes.txt`.
2. Проверка SGID: группа файла = `opslab`.
3. ACL: `guest1` = read на `notes.txt` **но** без доступа в каталог — зафиксируйте фактическое поведение (часто нужен `x` на каталог только для traverse или отдельная схема).  
   **Требование mid:** сделайте так, чтобы `guest1` мог **только читать** `notes.txt` (минимально необходимые права на путь) и **не мог писать**. Evidence обязателен.

### 4. Verify — негативы

От `su - guest1` / `ops1`:

1. Запись в `notes.txt` от guest → отказ.
2. `ops1` читает/пишет по группе.
3. Sticky: `/srv/opslab/tmp` `1770`; удаление чужого файла → отказ.

Всё → `~/lab-notes/03-evidence.txt`.

### 5. Change — sudoers drop-in

`/etc/sudoers.d/ops1` (имена unit: `cron` **или** `crond` под вашу семью):

```text
ops1 ALL=(root) NOPASSWD: /bin/systemctl restart cron, /bin/systemctl restart crond, /bin/systemctl status cron, /bin/systemctl status crond
```

Негатив: `ops1` **не** может `sudo id` / `sudo -i`.

### 6. Automate

`/usr/local/bin/lab-check-opslab.sh` (root): группа, пользователи, mode `/srv/opslab`, наличие ACL, синтаксис sudoers (`visudo -c`). Exit 0/1.

### 7. Document

В `03.md` (коротко, жёстко): угрозы модели; почему не «все в sudo»; чем ACL лучше раздувания группы.

## Критерии приёмки

- [ ] `dev1`/`ops1`/`guest1` и `/srv/opslab` 2770
- [ ] ACL read-only для guest доказан; запись запрещена
- [ ] Sticky-отказ показан
- [ ] Ограниченный sudo + негатив полного sudo
- [ ] Check-скрипт exit 0

## Подсказки

- Новые группы видны после нового login/`su -`.
- Только `visudo`.

## Очистка (опционально)

`guest1` можно удалить после проверок; `dev1`/`ops1` оставьте для следующих модулей.
