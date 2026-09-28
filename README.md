# Администрирование Linux — mid+ практический курс

Последовательный operator-first курс: bootstrap mid-уровня → система/сеть/службы → MAC → метрики/алерты → Ansible IaC → капстоун с handoff и postmortem.

Целевая планка: **middle+ / junior→middle transition** (не развлекательный junior-track). Ожидается базовая уверенность в CLI; модули 01–04 — сжатый bootstrap с жёсткими критериями, не «первые шаги в Linux».

Структура опирается на lab-driven курсы:

- этапы: bootstrap → система → сеть/защита → службы → MAC → observability → IaC → капстоун;
- цикл практики: **Inspect → Change → Verify → Document → Automate**;
- в каждом модуле — теория, задания, критерии приёмки (единый шаблон lab).

## Быстрый старт

1. Прочитайте [как учиться](course/how-to-study.md).
2. Поднимите лабораторию по [lab-setup](course/lab-setup.md).
3. Идите по [syllabus](course/syllabus.md) по порядку (капстоун **12** — после модулей **13–15**).
4. Финал — [капстоун](modules/12-capstone/lab.md).

## Программа (15 модулей)

| № | Модуль | Фокус |
|---|--------|--------|
| 01 | [Окружение и основы](modules/01-lab-fundamentals/) | Инвентарь стенда, FHS, man, dual-distro |
| 02 | [Оболочка и текст](modules/02-shell-text/) | bash, triage логов, grep/sed/awk |
| 03 | [Пользователи и права](modules/03-users-permissions/) | least privilege, sudo, ACL, негативные тесты |
| 04 | [Пакеты и обновления](modules/04-packages/) | apt/dnf, аудит, hold/pin, гигиена репо |
| 05 | [Диски и LVM](modules/05-storage-lvm/) | разделы, ФС, LVM, mount |
| 06 | [Процессы и systemd](modules/06-processes-systemd/) | ps, journal, unit, cron/timer |
| 07 | [Сеть и firewall](modules/07-networking-firewall/) | IP, DNS, nftables/firewalld |
| 08 | [SSH и базовая безопасность](modules/08-ssh-hardening/) | ключи, sshd, fail2ban |
| 09 | [Журналы и мониторинг](modules/09-logging-monitoring/) | journald, rsyslog, health-check → мост к 14 |
| 10 | [Службы: веб и обмен](modules/10-services-web-share/) | nginx + **TLS must**, NFS |
| 11 | [Скрипты и бэкапы / DR](modules/11-bash-backups/) | off-host target, timed restore, RPO/RTO |
| 13 | [MAC: SELinux / AppArmor](modules/13-mac-selinux-apparmor/) | отказ → AVC/логи → fix |
| 14 | [Observability: metrics + alerts](modules/14-observability-metrics/) | exporter + Prometheus lite, alert drill |
| 15 | [Ansible IaC](modules/15-ansible-iac/) | inventory, playbooks, check mode, handlers |
| 12 | [Капстоун](modules/12-capstone/) | mid+ стенд: TLS, metrics, Ansible, DR, postmortem |

Ориентир по времени: **55–80 часов** (без учёта повторения сложных лабораторий).

Нумерация: модули **13–15** идут **перед** капстоуном **12** (исторический номер капстоуна сохранён).

## Окружение

- **Debian-семейство:** Debian 12 или Ubuntu 22.04/24.04 LTS (AppArmor)
- **RHEL-семейство:** Rocky Linux 9 или AlmaLinux 9 (SELinux)
- Гипервизор: VirtualBox, VMware, KVM/libvirt или облачная ВМ
- **Обязательно 2 ВМ:** `srv` (сервер) и `cli` (клиент, DR-target, проверки)

## OpenSpec

Проект инициализирован с [OpenSpec](https://openspec.dev/):

- конфиг: `openspec/config.yaml`
- спецификации структуры курса: `openspec/specs/`
- команды Cursor: `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, …

## За пределами MVP (backlog)

Не входят в текущий mid+ MVP: HA/keepalived, SSSD/LDAP full, Kubernetes, глубокий perf/BPF (ориентиры модулей 16–23).

## Лицензия

Учебные материалы — MIT (см. `LICENSE`).
