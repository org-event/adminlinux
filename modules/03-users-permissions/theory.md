# Теория — пользователи и права

## Модель доступа

Каждый процесс выполняется от имени пользователя. Файлы имеют владельца, группу и режим доступа:

```text
rwxr-x---  user  group  file
```

| Символ | Для кого | Значение |
|--------|----------|----------|
| rwx | owner | чтение/запись/исполнение |
| r-x | group | чтение/исполнение |
| --- | other | нет доступа |

```bash
ls -l file
stat file
chmod 640 file
chmod u=rw,g=r,o= file
chown user:group file
```

## Учётные записи

```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
sudo groupadd developers
sudo usermod -aG developers alice
id alice
getent passwd alice
getent group developers
```

Файлы: `/etc/passwd`, `/etc/shadow`, `/etc/group`.

## sudo

```bash
sudo -l
sudo visudo
# пример: alice ALL=(ALL) /usr/bin/systemctl
```

Не раздавайте полный `ALL=(ALL) ALL` без необходимости.

## ACL (когда chmod мало)

```bash
sudo apt install acl    # Debian, если нет
# sudo dnf install acl  # RHEL
sudo setfacl -m u:alice:rw file
getfacl file
```

## Специальные биты (кратко)

- `SUID/SGID` — осторожно, понимайте зачем;
- sticky bit на `/tmp` (`chmod +t`) — удалять может владелец файла.
