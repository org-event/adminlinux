# Syllabus — mid+ последовательная программа

Курс из **15 модулей**. Капстоун **12** сдаётся **после** модулей **13–15**.

Планка: **middle+** (сжатый bootstrap 01–04, must: TLS, MAC, metrics/alerts, Ansible, off-host DR).

```text
Этап A  Bootstrap (mid) ..... 01 → 02 → 03 → 04
Этап B  Система ............. 05 → 06
Этап C  Сеть и защита ....... 07 → 08 → 09
Этап D  Службы и DR ......... 10 (TLS) → 11 (off-host DR)
Этап E  Mid+ ops ............ 13 (MAC) → 14 (metrics) → 15 (Ansible)
Этап F  Итог ................ 12 (капстоун)
```

## Модули

| Модуль | Название | Результат | Зависит от |
|--------|----------|-----------|------------|
| [01](../modules/01-lab-fundamentals/) | Окружение и основы | Инвентарь стенда, dual-distro паспорт | lab-setup |
| [02](../modules/02-shell-text/) | Оболочка и текст | Triage логов/конфигов CLI | 01 |
| [03](../modules/03-users-permissions/) | Пользователи и права | Least privilege + негативные тесты | 02 |
| [04](../modules/04-packages/) | Пакеты и обновления | Аудит, hold/pin, гигиена репо | 03 |
| [05](../modules/05-storage-lvm/) | Диски и LVM | Том, ФС, автомонтирование | 04 |
| [06](../modules/06-processes-systemd/) | Процессы и systemd | unit, timer, journal | 05 |
| [07](../modules/07-networking-firewall/) | Сеть и firewall | IP/DNS, правила firewall | 06 |
| [08](../modules/08-ssh-hardening/) | SSH и hardening | Ключи, hardened sshd | 07 |
| [09](../modules/09-logging-monitoring/) | Журналы и health-check | Центральный просмотр + мост к 14 | 08 |
| [10](../modules/10-services-web-share/) | Веб и обмен файлами | nginx + **HTTPS**, NFS | 07–09 |
| [11](../modules/11-bash-backups/) | Скрипты и DR | Off-host бэкап, timed restore, RPO/RTO | 05–10 |
| [13](../modules/13-mac-selinux-apparmor/) | MAC: SELinux / AppArmor | Отказ → логи → fix (не disable навсегда) | 08–10 |
| [14](../modules/14-observability-metrics/) | Metrics + alerts | Exporter + Prometheus lite, alert drill | 09–10 |
| [15](../modules/15-ansible-iac/) | Ansible IaC | Playbooks srv+cli, check mode, handlers | 08–11, 10 |
| [12](../modules/12-capstone/) | Капстоун | Mid+ стенд + postmortem + handoff | 01–11, 13–15 |

## Критерии готовности курса (для автора/ревьюера)

- [ ] У каждого модуля есть `README.md`, `theory.md`, `lab.md`, `checklist.md`
- [ ] Лаборатории проверяемы без преподавателя; шаблон: Цель / Окружение / Задания / Критерии / Подсказки / Очистка
- [ ] Dual-distro (Debian/Ubuntu vs Rocky/Alma) там, где команды расходятся
- [ ] Капстоун must: TLS, metrics/alerts, Ansible-built, DR drill, postmortem
- [ ] `course/lab-setup.md` описывает обе семьи дистрибутивов и 2 ВМ
