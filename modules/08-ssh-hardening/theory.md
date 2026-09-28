# Теория — SSH и hardening

## Ключи вместо пароля

На клиенте:

```bash
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_lab
ssh-copy-id -i ~/.ssh/id_ed25519_lab.pub user@srv
ssh -i ~/.ssh/id_ed25519_lab user@srv
```

На сервере ключ попадает в `~/.ssh/authorized_keys` (права на `.ssh` = `700`, на файл = `600`).

## Базовый hardening sshd

Идеи (включайте **по одной**, проверяя новую сессию до закрытия старой):

- `PasswordAuthentication no` (после проверки ключей)
- `PermitRootLogin no`
- `AllowUsers ops1 youruser` (опционально)
- смена порта — спорная мера; важнее ключи и firewall

Проверка конфига:

```bash
sudo sshd -t
sudo systemctl reload ssh    # или sshd
```

## fail2ban / подобное

Бан по повторным неудачным попыткам SSH снижает шум ботов. На учебных стендах с белым IP польза меньше, но навык полезен.

## Не делайте

- Не отключайте пароль, пока второе окно с ключом не подтверждено.
- Не редактируйте `sshd_config` без бэкапа.
- Не закрывайте консоль гипервизора в момент эксперимента.
