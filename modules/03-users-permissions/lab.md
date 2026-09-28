# Лаборатория 03 — пользователи и права

## Цель

Собрать учебную модель доступа: группа проекта, общий каталог, ограниченный sudo.

## Окружение

- ВМ `srv`
- Снимок перед началом рекомендуется

## Задания

### 1. Inspect

```bash
id
getent passwd | tail -5
ls -ld /home
sudo -l
```

### 2. Change — пользователи и группа

Создайте:

- группу `opslab`
- пользователей `dev1` и `ops1` (с домашними каталогами и bash)
- `dev1` и `ops1` в группе `opslab`
- каталог `/srv/opslab` с владельцем `root:opslab` и правами `2770` (SGID на каталоге)

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

Задайте пароли учебные и запишите их только в локальные заметки стенда.

### 3. Change — общий файл и ACL

1. От пользователя `dev1` создайте `/srv/opslab/notes.txt`.
2. Проверьте, что `ops1` может читать/писать благодаря группе.
3. Создайте пользователя `guest1` **без** группы `opslab`.
4. Через ACL выдайте `guest1` только чтение `notes.txt` (не запись).

### 4. Change — ограниченный sudo

Разрешите `ops1` перезапускать `cron`/`crond` без полного root (через `/etc/sudoers.d/ops1`).

Пример идеи (адаптируйте имя unit/сервиса под дистрибутив):

```text
ops1 ALL=(root) NOPASSWD: /bin/systemctl restart cron, /bin/systemctl restart crond, /bin/systemctl status cron, /bin/systemctl status crond
```

Проверьте синтаксис: `sudo visudo -c`.

### 5. Verify

От разных пользователей покажите:

```bash
# как guest1 — нет входа в /srv/opslab листингом? или нет записи в notes
# как ops1 — sudo systemctl status cron/crond работает
# getfacl /srv/opslab/notes.txt
```

Сохраните доказательства в `~/lab-notes/03-evidence.txt` (копируйте выводы).

### 6. Document

В `~/lab-notes/03.md` объясните, зачем `2770` на каталоге проекта.

## Критерии приёмки

- [ ] Пользователи `dev1`, `ops1`, `guest1` существуют
- [ ] `/srv/opslab` имеет группу `opslab` и режим с SGID
- [ ] `guest1` читает файл по ACL, но не пишет
- [ ] `ops1` имеет ограниченный sudo на systemctl для cron/crond
- [ ] Есть файл доказательств

## Подсказки

- Новые группы применяются после перелогина: `su - user` или новый SSH-сеанс.
- Ошибки sudoers чинятся с `visudo`; не редактируйте sudoers «вслепую» под root без проверки.

## Очистка (опционально)

```bash
sudo userdel -r guest1   # когда закончите проверки
# остальных лучше оставить для следующих модулей
```
