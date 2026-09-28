# Syllabus — mid+ последовательная программа

Курс из **20 учебных модулей** + капстоун **12**. Капстоун сдаёте **после** **13–15** и эшелона **16–20**.

Планка: **middle+**. Сжатый bootstrap 01–04; обязательно: TLS, MAC, метрики/алерты, Ansible, DR на другой хост. Эшелон: perf, identity, HA, контейнеры, продвинутая сеть.

```text
Этап A  Bootstrap (mid) ..... 01 → 02 → 03 → 04
Этап B  Система ............. 05 → 06
Этап C  Сеть и защита ....... 07 → 08 → 09
Этап D  Службы и DR ......... 10 (TLS) → 11 (DR на другой хост)
Этап E  Mid+ ops ............ 13 (MAC) → 14 (метрики) → 15 (Ansible)
Этап F  Post-MVP ops ........ 16 (perf) → 17 (identity) → 18 (HA)
                              → 19 (контейнеры) → 20 (сеть)
Этап G  Итог ................ 12 (капстоун)
```

Почему так: identity опирается на users/SSH/MAC; контейнеры — после systemd и Ansible; HA и продвинутая сеть — после базовой сети и веб-службы; perf — после процессов и метрик; капстоун собирает всё.

## Модули

| Модуль | Название | Результат | Зависит от |
|--------|----------|-----------|------------|
| [01](../modules/01-lab-fundamentals/) | Окружение и основы | Инвентарь стенда, паспорт двух семей | lab-setup |
| [02](../modules/02-shell-text/) | Оболочка и текст | Разбор логов и конфигов в CLI | 01 |
| [03](../modules/03-users-permissions/) | Пользователи и права | Least privilege + проверки отказа | 02 |
| [04](../modules/04-packages/) | Пакеты и обновления | Аудит, hold/pin, гигиена репозиториев | 03 |
| [05](../modules/05-storage-lvm/) | Диски и LVM | Том, ФС, автомонтирование | 04 |
| [06](../modules/06-processes-systemd/) | Процессы и systemd | unit, timer, journal | 05 |
| [07](../modules/07-networking-firewall/) | Сеть и firewall | IP/DNS, правила firewall | 06 |
| [08](../modules/08-ssh-hardening/) | SSH и hardening | Ключи, ужесточённый sshd | 07 |
| [09](../modules/09-logging-monitoring/) | Журналы и health-check | Центральный просмотр + мост к 14/16 | 08 |
| [10](../modules/10-services-web-share/) | Веб и обмен файлами | nginx + **HTTPS**, NFS | 07–09 |
| [11](../modules/11-bash-backups/) | Скрипты и DR | Бэкап на другой хост, timed restore, RPO/RTO | 05–10 |
| [13](../modules/13-mac-selinux-apparmor/) | MAC: SELinux / AppArmor | Отказ → логи → исправление (не disable навсегда) | 08–10 |
| [14](../modules/14-observability-metrics/) | Метрики и алерты | Exporter + Prometheus lite, drill по алерту | 09–10 |
| [15](../modules/15-ansible-iac/) | Ansible IaC | Playbooks srv+cli, check mode, handlers | 08–11, 10 |
| [16](../modules/16-performance-capacity/) | Performance и capacity | Baseline, деградация, пороги, заметки по capacity | 06, 09, 14 |
| [17](../modules/17-identity-sssd/) | Identity: SSSD + LDAP | Центральный логин, getent, break-glass | 03, 08 (13) |
| [18](../modules/18-ha-reliability/) | HA lite | VIP/keepalived или failover upstream + SPOF | 07, 10 |
| [19](../modules/19-containers-orchestration/) | Контейнеры + k8s lite | Podman/Docker + Deploy/Service/probes | 06, 07, 15 |
| [20](../modules/20-advanced-networking/) | Продвинутая сеть | DNS как служба, WireGuard, зоны/egress | 07, 08 |
| [12](../modules/12-capstone/) | Капстоун | Стенд mid+ + усиление из эшелона + postmortem | 01–11, 13–20 |

## Критерии готовности курса (для автора/ревьюера)

- [ ] У каждого модуля есть `README.md`, `theory.md`, `lab.md`, `checklist.md`
- [ ] Лаборатории проверяемы без преподавателя; шаблон: Цель / Окружение / Задания / Критерии / Подсказки / Очистка
- [ ] Две семьи (Debian/Ubuntu vs Rocky/Alma) там, где команды расходятся
- [ ] Капстоун обязательно: TLS, метрики/алерты, сборка через Ansible, DR-drill, postmortem
- [ ] Капстоун: ≥1 пункт из HA / identity / containers (обязательно или на выбор) со ссылками на 16–20
- [ ] `course/lab-setup.md` описывает обе семьи дистрибутивов и 2 ВМ
