# Администрирование Linux — mid+ практический курс

Последовательный operator-first курс: bootstrap mid-уровня → система/сеть/службы → MAC → метрики/алерты → Ansible IaC → эшелон ops (perf, identity, HA, containers, advanced net) → капстоун с handoff и postmortem.

Целевая планка: **middle+ / junior→middle transition** (не развлекательный junior-track). Ожидается базовая уверенность в CLI; модули 01–04 — сжатый bootstrap с жёсткими критериями, не «первые шаги в Linux».

Структура опирается на lab-driven курсы:

- этапы: bootstrap → система → сеть/защита → службы → MAC → observability → IaC → post-MVP ops → капстоун;
- цикл практики: **Inspect → Change → Verify → Document → Automate**;
- в каждом модуле — теория, задания, критерии приёмки (единый шаблон lab).

## Быстрый старт

1. Прочитайте [как учиться](course/how-to-study.md).
2. Поднимите лабораторию по [lab-setup](course/lab-setup.md).
3. Идите по [syllabus](course/syllabus.md) по порядку (капстоун **12** — после **13–15** и **16–20**).
4. Финал — [капстоун](modules/12-capstone/lab.md).

## Программа (20 модулей + капстоун 12)

| № | Модуль | Фокус |
|---|--------|--------|
| 01 | [Окружение и основы](modules/01-lab-fundamentals/) | Инвентарь стенда, FHS, man, dual-distro |
| 02 | [Оболочка и текст](modules/02-shell-text/) | bash, triage логов, grep/sed/awk |
| 03 | [Пользователи и права](modules/03-users-permissions/) | least privilege, sudo, ACL → мост к 17 |
| 04 | [Пакеты и обновления](modules/04-packages/) | apt/dnf, аудит, hold/pin, гигиена репо |
| 05 | [Диски и LVM](modules/05-storage-lvm/) | разделы, ФС, LVM, mount |
| 06 | [Процессы и systemd](modules/06-processes-systemd/) | ps, journal, unit, cron/timer |
| 07 | [Сеть и firewall](modules/07-networking-firewall/) | IP, DNS, nftables/firewalld → мост к 20 |
| 08 | [SSH и базовая безопасность](modules/08-ssh-hardening/) | ключи, sshd, fail2ban |
| 09 | [Журналы и мониторинг](modules/09-logging-monitoring/) | journald, health-check → мост к 14/16 |
| 10 | [Службы: веб и обмен](modules/10-services-web-share/) | nginx + **TLS must**, NFS |
| 11 | [Скрипты и бэкапы / DR](modules/11-bash-backups/) | off-host target, timed restore, RPO/RTO |
| 13 | [MAC: SELinux / AppArmor](modules/13-mac-selinux-apparmor/) | отказ → AVC/логи → fix |
| 14 | [Observability: metrics + alerts](modules/14-observability-metrics/) | exporter + Prometheus lite, alert drill |
| 15 | [Ansible IaC](modules/15-ansible-iac/) | inventory, playbooks, check mode, handlers |
| 16 | [Performance и capacity](modules/16-performance-capacity/) | baseline, load vs pressure, пороги |
| 17 | [Identity: SSSD + LDAP](modules/17-identity-sssd/) | центральный логин, getent, break-glass |
| 18 | [HA и надёжность](modules/18-ha-reliability/) | VIP/keepalived или upstream failover |
| 19 | [Контейнеры + k8s lite](modules/19-containers-orchestration/) | Podman/Docker + Deploy/Service/probes |
| 20 | [Продвинутая сеть](modules/20-advanced-networking/) | DNS-as-service, WireGuard, zones/egress |
| 12 | [Капстоун](modules/12-capstone/) | mid+ стенд + усиление из 16–20 + postmortem |

**Порядок прохождения:** `01→11 → 13→15 → 16→20 → 12`.

Ориентир по времени: **75–110 часов** (MVP ~55–80 + эшелон 16–20).

Нумерация: модули **13–20** идут **перед** капстоуном **12** (исторический номер капстоуна сохранён).

## Окружение

- **Debian-семейство:** Debian 12 или Ubuntu 22.04/24.04 LTS (AppArmor)
- **RHEL-семейство:** Rocky Linux 9 или AlmaLinux 9 (SELinux)
- Гипервизор: VirtualBox, VMware, KVM/libvirt или облачная ВМ
- **Обязательно 2 ВМ:** `srv` (сервер) и `cli` (клиент, DR-target, проверки)
- Для 18/19 иногда удобна третья лёгкая ВМ или больше RAM на `srv` (k3s/kind)

## OpenSpec

Проект инициализирован с [OpenSpec](https://openspec.dev/):

- конфиг: `openspec/config.yaml`
- спецификации структуры курса: `openspec/specs/`
- команды Cursor: `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, …

## За пределами текущего эшелона (backlog)

Позже можно добавить: полный FreeIPA/replicas, Pacemaker/Corosync, CKA-depth, eBPF/perf deep-dive, CI для Ansible, multi-site DR, IdM MFA, service mesh.

## Лицензия

Учебные материалы — MIT (см. `LICENSE`).
