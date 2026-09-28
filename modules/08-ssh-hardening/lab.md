# Лаборатория 08 — SSH и hardening

## Цель

Настроить вход по ключу, ужесточить sshd безопасным порядком шагов и проверить негативными тестами (пароль, root, права на ключи).

## Окружение

- `srv` + машина-клиент (`cli` или ваш хост)
- Снимок `before-ssh-hardening`
- **Консоль гипервизора открыта**

## Задания

### 1. Inspect — текущая политика sshd

```bash
sudo systemctl status ssh || sudo systemctl status sshd
sudo sshd -T | egrep 'passwordauthentication|permitrootlogin|pubkeyauthentication|kbdinteractiveauthentication|maxauthtries|allowusers|allowgroups'
ls -la ~/.ssh 2>/dev/null || true
```

Скопируйте конфиг:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak.$(date +%F)
sudo mkdir -p /etc/ssh/sshd_config.d
```

Сохраните «до» в `~/lab-notes/08-sshd-before.txt` (`sshd -T` выборка).

### 2. Change — ключи (безопасный порядок)

1. На клиенте создайте ключ ed25519 (отдельный учебный файл, с comment):

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_lab -C "lab-admin@$(hostname)"
```

2. Установите pubkey на `srv` (`ssh-copy-id` или вручную в `~/.ssh/authorized_keys`).
3. Проверьте права: `~/.ssh` → `700`, `authorized_keys` → `600`.
4. Войдите **второй** сессией только по ключу.
5. Только после этого переходите к отключению паролей.

### 3. Verify — негативный тест прав на ключи

1. На `srv` временно выставьте `chmod 777 ~/.ssh` или `666 authorized_keys`.
2. Попробуйте войти по ключу — sshd часто **отклонит** ключ. Зафиксируйте в `~/lab-notes/08-keyperms.txt`.
3. Верните `700`/`600` и подтвердите вход.

### 4. Change — sshd hardening

Файл `/etc/ssh/sshd_config.d/99-lab-hardening.conf` (если поддерживается drop-in):

```text
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
MaxAuthTries 3
# опционально для учебной ВМ с одним админом:
# AllowUsers youruser
```

```bash
sudo sshd -t
sudo systemctl reload ssh || sudo systemctl reload sshd
```

Сохраните «после»: `sshd -T` в `~/lab-notes/08-sshd-after.txt`.

### 5. Verify — негативные тесты доступа

С клиента:

```bash
# пароль должен быть отвергнут
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@srv

# ключ должен пустить
ssh -i ~/.ssh/id_ed25519_lab user@srv

# root по SSH — отказ (если PermitRootLogin no)
ssh -i ~/.ssh/id_ed25519_lab root@srv || true
```

Все выводы — в `~/lab-notes/08-evidence.txt`.

### 6. Change — banner и сессионные параметры (практика)

1. Создайте `/etc/ssh/lab-banner.txt` с предупреждением «учебный стенд».
2. Добавьте в drop-in: `Banner /etc/ssh/lab-banner.txt`.
3. Опционально: `ClientAliveInterval 60` и `ClientAliveCountMax 3`.
4. `sshd -t` + reload; при новом логине banner виден.

### 7. Change (рекомендуется) — fail2ban

| Семья | Установка |
|-------|-----------|
| Debian/Ubuntu | `sudo apt install fail2ban` |
| Rocky/Alma | `sudo dnf install fail2ban` |

Включите jail для ssh, покажите:

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd || sudo fail2ban-client status ssh
```

### 8. Automate — проверка hardening одной командой

Скрипт `/usr/local/bin/lab-ssh-audit.sh` на `srv`:

- печатает `sshd -T` по ключевым параметрам;
- проверяет, что `passwordauthentication no`, `permitrootlogin no`;
- код 0 если политика ок, иначе 1.

### 9. Document

Runbook в `~/lab-notes/08.md`: «закрыли себе SSH — восстановление через консоль» (single user / login на консоли → правка drop-in → `sshd -t` → reload).

## Критерии приёмки

- [ ] Вход по ключу работает
- [ ] Вход паролем отключён (негативный тест)
- [ ] Root login по SSH запрещён (негативный тест)
- [ ] Негативный тест прав `~/.ssh` выполнен и откатан
- [ ] Есть бэкап sshd_config, before/after, evidence
- [ ] Есть скрипт audit и runbook восстановления
- [ ] (Рекомендуется) fail2ban показывает jail ssh

## Подсказки

- Держите старую SSH-сессию открытой до проверки новой.
- Имя сервиса: `ssh` (Debian) vs `sshd` (RHEL).
- Не отключайте пароли, пока ключ не проверен второй сессией.

## Очистка

Состояние hardened — целевое. Откат только снимок/консоль.
