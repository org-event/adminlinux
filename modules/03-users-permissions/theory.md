# Теория — доступ (mid)

## Модель

Владелец / группа / other + (по желанию) ACL. SGID на каталоге наследует группу для новых файлов. Sticky (`+t`) — нельзя удалить чужой файл в общем tmp.

```bash
chmod 2770 /srv/opslab          # SGID + rwx для group
chmod 1770 /srv/opslab/tmp      # sticky
setfacl -m u:guest1:r -- file
getfacl file
```

## sudo

Только `/etc/sudoers.d/` через `visudo -f` + `visudo -c`. Полный `ALL=(ALL) ALL` — долг, не норма на mid+.

## Учётки

```bash
useradd -m -s /bin/bash name
usermod -aG group name
getent passwd name; id name
```

Файлы истины: `/etc/passwd`, `/etc/shadow`, `/etc/group`.
