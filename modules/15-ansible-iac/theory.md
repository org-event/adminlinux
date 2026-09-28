# Теория — Ansible для mid+ стенда

## Зачем

Ручной стенд дрейфует. IaC фиксирует желаемое состояние: повторный apply должен быть идемпотентным; `check` показывает drift.

## Минимальная раскладка курса

```text
~/ansible-lab/
  ansible.cfg
  inventory/hosts.ini      # srv, cli
  group_vars/all.yml
  playbooks/
    site.yml               # импорт остальных
    packages.yml
    users.yml
    sshd.yml
    firewall.yml
    nginx_tls.yml
    timers.yml
  files/  templates/
```

## Ключевые приёмы

```bash
ansible all -m ping
ansible-playbook playbooks/site.yml --check --diff
ansible-playbook playbooks/site.yml
```

- **Handlers** — `notify: reload nginx` / `restart sshd` только при change.
- **Drop-in** для sshd/nginx предпочтительнее править весь файл целиком вслепую.
- Dual-distro: `ansible_os_family` / отдельные vars для `apt` vs `dnf`, `ufw` vs `firewalld`.

## Безопасность

Секреты — в ansible-vault или вне git; учебные пароли не коммитить в публичный курс-репозиторий.
