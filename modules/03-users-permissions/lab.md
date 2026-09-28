# Лаборатория 03 — пользователи и права

## Цель

Собрать учебную модель доступа: группа проекта, общий каталог, ACL, ограниченный sudo; проверить отказами.

## Окружение

- ВМ `srv`
- Снимок перед началом рекомендуется

## Задания

### 1. Inspect

```bash
id
getent passwd | tail -5
getent group
ls -ld /home
sudo -l
umask
```

Запишите текующий `umask` в заметки.

### 2. Change — пользователи и группа

Создайте:

- группу `opslab`
- пользователей `dev1` и `ops1` (домашние + bash)
- обоих в `opslab`
- `/srv/opslab` с `root:opslab` и `2770` (SGID)

```bash
sudo groupadd opslab
sudo useradd -m -s /bin/bash dev1
sudo useradd -m -s /bin/bash ops1
sudo usermod -aG opslab dev1
sudo usermod -aG opslab ops1
sudo mkdir -p /srv/opslab
sudo chown root:opslab /srv/opslab
sudo chmod 2770 /srv/opslab
```

Учебные пароли — только в локальные заметки стенда.

### 3. Change — общий файл, SGID, ACL

1. От `dev1` создайте `/srv/opslab/notes.txt`.
2. Проверьте: новый файл наследует группу `opslab` (SGID).
3. `ops1` читает/пишет благодаря группе.
4. Создайте `guest1` **без** `opslab`.
5. ACL: `guest1` — только чтение `notes.txt`.

```bash
sudo setfacl -m u:guest1:r -- /srv/opslab/notes.txt
getfacl /srv/opslab/notes.txt
```

### 4. Verify — негативные тесты доступа

От разных пользователей (через `su -`):

1. `guest1` — **не** может `ls`/`cd` в `/srv/opslab`? или не может писать в `notes` — зафиксируйте фактическое поведение.
2. `guest1` — попытка записи в `notes.txt` должна провалиться.
3. Посторонний файл: создайте `/srv/opslab/secret.env` mode `640`, убедитесь что `guest1` не читает даже при желании «угадать путь», если каталог недоступен.

Сохраните выводы в `~/lab-notes/03-evidence.txt`.

### 5. Change — ограниченный sudo

`/etc/sudoers.d/ops1` (через `visudo -f`):

```text
ops1 ALL=(root) NOPASSWD: /bin/systemctl restart cron, /bin/systemctl restart crond, /bin/systemctl status cron, /bin/systemctl status crond
```

```bash
sudo visudo -c
sudo visudo -f /etc/sudoers.d/ops1
```

Проверка: `ops1` может `sudo systemctl status cron|crond`, но **не** может `sudo id` / `sudo bash` (негативный тест).

### 6. Change — sticky bit (демо)

```bash
sudo mkdir -p /srv/opslab/tmp-sticky
sudo chmod 1770 /srv/opslab/tmp-sticky
# от dev1 и ops1 создайте по файлу; попробуйте удалить чужой — ожидайте отказ
```

Кратко в заметках: зачем sticky на `/tmp`.

### 7. Automate — проверка модели доступа

Скрипт `/usr/local/bin/lab-check-opslab.sh` (root): проверяет существование группы/пользователей, режим `/srv/opslab`, наличие ACL на `notes.txt`. Exit 0/1.

### 8. Document

В `~/lab-notes/03.md`: зачем `2770`; чем ACL лучше «добавить всех в группу»; риск полного sudo у `ops1`.

## Критерии приёмки

- [ ] Пользователи `dev1`, `ops1`, `guest1` существуют
- [ ] `/srv/opslab` — группа `opslab` + SGID
- [ ] `guest1` читает по ACL, не пишет; негативные тесты зафиксированы
- [ ] `ops1` — ограниченный sudo; полный sudo не работает
- [ ] Показан sticky-bit отказ на чужой файл
- [ ] Есть evidence и check-скрипт

## Подсказки

- Новые группы — после `su - user` / нового SSH.
- sudoers только через `visudo`.

## Очистка (опционально)

```bash
sudo userdel -r guest1   # когда закончите проверки
# остальных лучше оставить для следующих модулей
```
