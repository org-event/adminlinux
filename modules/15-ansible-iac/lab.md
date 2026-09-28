# Лаборатория 15 — Ansible IaC

## Цель

Собрать inventory и playbooks, которые поднимают/сверяют ключевые mid+ настройки. Докажите check mode, handlers и идемпотентность.

## Окружение

- Управляющий узел: `cli` **или** ваш хост с SSH к обеим ВМ (предпочтительно playbooks гонять с `cli`)
- Ansible установлен (`apt/dnf install ansible` или pipx)
- Доступ по ключу к `srv` и (для ping) к `cli` localhost/`cli`

## Задания

### 1. Подготовьте каркас

```bash
ansible --version
mkdir -p ~/ansible-lab/{inventory,playbooks,files,templates,group_vars}
```

Инвентарь `inventory/hosts.ini`:

```ini
[servers]
srv ansible_host=<IP-srv>

[clients]
cli ansible_host=<IP-cli>

[lab:children]
servers
clients
```

`ansible lab -m ping` → пакет доказательств `15-ping.txt`.

### 2. Напишите — packages.yml

- Пакеты: `curl`, `jq`, `nginx` (на srv), полезные утилиты.
- Ветвление `ansible_os_family == "Debian"` vs `"RedHat"`.

### 3. Напишите — users.yml

- Группа `opslab`, пользователи `ops1`/`dev1` (или сверка существования).
- Не хардкодьте пароли в plain yaml в git; используйте `password` из vault **или** только ключи/`createhome` без пароля + заметка.

### 4. Напишите — sshd.yml

- Drop-in `/etc/ssh/sshd_config.d/99-lab-hardening.conf`: `PasswordAuthentication no`, `PermitRootLogin no`, … (согласовано с модулем 08).
- Handler: `reload/restart sshd`.
- **Важно:** не отрежьте себя — проверьте Console/снимок; сначала `--check`, потом apply при активной ключ-сессии. Если не вышло — откатитесь со снимка и повторите с открытой консолью.

### 5. Напишите — firewall.yml

- Разрешите SSH, HTTPS, NFS (lab), node_exporter только с IP cli.
- Модули/команды под ufw **или** firewalld.

### 6. Напишите — nginx_tls.yml

- Корень сайта `/var/www/lab`, index из template.
- TLS crt/key: генерировать `openssl` task **или** копировать из `files/` (учебные).
- `notify: reload nginx`; validate `nginx -t` перед reload (careful с handlers).

### 7. Напишите — timers.yml

- Unit+timer lab-backup (или deploy файлов из модуля 11) на srv.
- `daemon-reload` handler.

### 8. Соберите — site.yml

Импорт playbooks в разумном порядке. Теги (`packages`, `sshd`, …) — желательно.

### 9. Проверьте — check / apply / idempotency

```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml --check --diff | tee ~/lab-notes/15-check.txt
ansible-playbook -i inventory/hosts.ini playbooks/site.yml | tee ~/lab-notes/15-apply.txt
ansible-playbook -i inventory/hosts.ini playbooks/site.yml | tee ~/lab-notes/15-idempotent.txt
```

Во втором apply — минимум changed (идеал: 0 changed на стабильном стенде).

### 10. Проверьте — handlers

Внесите harmless change в template index → apply → убедитесь, что handler reload nginx сработал (в output `RUNNING HANDLER`).

### 11. Задокументируйте

`15.md`: схема ролей; как гонять check в CI мыслимо; drift если правили руками после apply; связь с капстоуном.

## Критерии приёмки

- [ ] Inventory srv+cli; ping OK
- [ ] Playbooks: packages, users, sshd, firewall, nginx+tls, timer (+ site)
- [ ] Dual-distro ветвление там, где нужно
- [ ] сохранён вывод `--check --diff`
- [ ] Apply + повторная идемпотентность
- [ ] Handler reload продемонстрирован
- [ ] HTTPS после apply отвечает с cli

## Подсказки

| Тема | Совет |
|------|-------|
| sshd | `validate: /usr/sbin/sshd -t -f %s` где применимо |
| firewalld | permanent + reload |
| SELinux | copy с правильным context или `sefcontext` — иначе модуль 13 |
| Секреты | не коммитьте ключи/пароли в adminlinux git |

## Очистка

Каталог `~/ansible-lab` оставьте для капстоуна; в course-artifacts можно положить **обезличенные** примеры без секретов.
