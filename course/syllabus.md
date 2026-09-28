# Syllabus — mid+ последовательная программа

Курс из **20 учебных модулей** + капстоун **12**. Капстоун сдаётся **после** **13–15** и эшелона **16–20**.

Планка: **middle+** (сжатый bootstrap 01–04, must: TLS, MAC, metrics/alerts, Ansible, off-host DR; эшелон: perf, identity, HA, containers, advanced net).

```text
Этап A  Bootstrap (mid) ..... 01 → 02 → 03 → 04
Этап B  Система ............. 05 → 06
Этап C  Сеть и защита ....... 07 → 08 → 09
Этап D  Службы и DR ......... 10 (TLS) → 11 (off-host DR)
Этап E  Mid+ ops ............ 13 (MAC) → 14 (metrics) → 15 (Ansible)
Этап F  Post-MVP ops ........ 16 (perf) → 17 (identity) → 18 (HA)
                              → 19 (containers) → 20 (adv net)
Этап G  Итог ................ 12 (капстоун)
```

Почему так: identity опирается на users/SSH/MAC; containers — после systemd и Ansible; HA и advanced net — после базовой сети и веб-службы; perf — после процессов и метрик; капстоун собирает всё.

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
| [09](../modules/09-logging-monitoring/) | Журналы и health-check | Центральный просмотр + мост к 14/16 | 08 |
| [10](../modules/10-services-web-share/) | Веб и обмен файлами | nginx + **HTTPS**, NFS | 07–09 |
| [11](../modules/11-bash-backups/) | Скрипты и DR | Off-host бэкап, timed restore, RPO/RTO | 05–10 |
| [13](../modules/13-mac-selinux-apparmor/) | MAC: SELinux / AppArmor | Отказ → логи → fix (не disable навсегда) | 08–10 |
| [14](../modules/14-observability-metrics/) | Metrics + alerts | Exporter + Prometheus lite, alert drill | 09–10 |
| [15](../modules/15-ansible-iac/) | Ansible IaC | Playbooks srv+cli, check mode, handlers | 08–11, 10 |
| [16](../modules/16-performance-capacity/) | Performance и capacity | Baseline, деградация, пороги, capacity notes | 06, 09, 14 |
| [17](../modules/17-identity-sssd/) | Identity: SSSD + LDAP | Центральный логин, getent, break-glass | 03, 08 (13) |
| [18](../modules/18-ha-reliability/) | HA lite | VIP/keepalived или upstream failover + SPOF | 07, 10 |
| [19](../modules/19-containers-orchestration/) | Containers + k8s lite | Podman/Docker + Deploy/Service/probes | 06, 07, 15 |
| [20](../modules/20-advanced-networking/) | Advanced networking | DNS-as-service, WireGuard, zones/egress | 07, 08 |
| [12](../modules/12-capstone/) | Капстоун | Mid+ стенд + эшелон-усиление + postmortem | 01–11, 13–20 |

## Критерии готовности курса (для автора/ревьюера)

- [ ] У каждого модуля есть `README.md`, `theory.md`, `lab.md`, `checklist.md`
- [ ] Лаборатории проверяемы без преподавателя; шаблон: Цель / Окружение / Задания / Критерии / Подсказки / Очистка
- [ ] Dual-distro (Debian/Ubuntu vs Rocky/Alma) там, где команды расходятся
- [ ] Капстоун must: TLS, metrics/alerts, Ansible-built, DR drill, postmortem
- [ ] Капстоун: ≥1 пункт из HA / identity / containers (must или should на выбор) со ссылками на 16–20
- [ ] `course/lab-setup.md` описывает обе семьи дистрибутивов и 2 ВМ
