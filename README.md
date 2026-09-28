# Администрирование Linux — практический курс

Последовательный курс с лабораториями: от первого входа до развёрнутого и защищённого мини-сервера.

Структура и педагогический подход опираются на открытые operator-first / lab-driven курсы:

- этапы: основы → CLI → пользователи → пакеты/хранилище → сеть/службы → безопасность → автоматизация;
- цикл практики: **Inspect → Change → Verify → Document → Automate**;
- в каждом модуле — теория, задания, критерии приёмки.

## Быстрый старт

1. Прочитайте [как учиться](course/how-to-study.md).
2. Поднимите лабораторию по [lab-setup](course/lab-setup.md).
3. Идите по [syllabus](course/syllabus.md) строго по порядку модулей.
4. В конце — [капстоун](modules/12-capstone/lab.md).

## Программа (12 модулей)

| № | Модуль | Фокус |
|---|--------|--------|
| 01 | [Окружение и основы](modules/01-lab-fundamentals/) | ВМ, вход, man, дерево ФС |
| 02 | [Оболочка и текст](modules/02-shell-text/) | bash, пути, grep/sed/awk |
| 03 | [Пользователи и права](modules/03-users-permissions/) | user/group, chmod, sudo, ACL |
| 04 | [Пакеты и обновления](modules/04-packages/) | apt/dnf, репозитории |
| 05 | [Диски и LVM](modules/05-storage-lvm/) | разделы, ФС, LVM, mount |
| 06 | [Процессы и systemd](modules/06-processes-systemd/) | ps, journal, unit, cron/timer |
| 07 | [Сеть и firewall](modules/07-networking-firewall/) | IP, DNS, nftables/firewalld |
| 08 | [SSH и базовая безопасность](modules/08-ssh-hardening/) | ключи, sshd, fail2ban |
| 09 | [Журналы и мониторинг](modules/09-logging-monitoring/) | journald, rsyslog, health-check |
| 10 | [Службы: веб и обмен](modules/10-services-web-share/) | nginx, NFS |
| 11 | [Скрипты и бэкапы](modules/11-bash-backups/) | bash, cron, rsync |
| 12 | [Капстоун](modules/12-capstone/) | полный стенд с handoff |

Ориентир по времени: **40–60 часов** (без учёта повторения сложных лабораторий).

## Окружение

- **Debian-семейство:** Debian 12 или Ubuntu 22.04/24.04 LTS  
- **RHEL-семейство:** Rocky Linux 9 или AlmaLinux 9  
- Гипервизор: VirtualBox, VMware, KVM/libvirt или облачная ВМ  
- Рекомендуется 2 ВМ: `srv` (сервер) и `cli` (клиент для проверок)

## OpenSpec

Проект инициализирован с [OpenSpec](https://openspec.dev/):

- конфиг: `openspec/config.yaml`
- спецификации структуры курса: `openspec/specs/`
- команды Cursor: `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, …

## Лицензия

Учебные материалы — MIT (см. `LICENSE`).
