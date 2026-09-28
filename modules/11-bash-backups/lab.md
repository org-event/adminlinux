# Лаборатория 11 — off-host DR

## Цель

Автобэкап критичных путей на второй target, ротация, checksum, timed restore drill с зафиксированными RPO/RTO.

## Окружение

- `srv` с данными модулей 05/10; **`cli` или второй диск**
- SSH-ключ `srv→cli` для rsync (если выбран сетевой target)
- Снимок перед правкой timer/скриптов

## Задания

### 1. Inspect — scope + политика чисел

`~/lab-notes/11-scope.txt` — что бэкапим (nginx+ssl, exports, sudoers.d lab, `/var/www/lab`, NFS share, lab units, манифест пакетов).

В `11.md` **обязательно числа**:

- RPO lab = **≤ 60 мин** (или жёстче, если timer чаще)
- RTO lab = **≤ 30 мин** на восстановление каталога сайта/share и проверку `curl -k https://…`

### 2. Change — второй target

Выберите и зафиксируйте в inventory:

| Вариант | Target |
|---------|--------|
| A (предпочтительно) | `cli:/var/backups/srv/` по `rsync -aH` + SSH |
| B | второй диск → `/mnt/backup-disk/srv/` |

Локальный `/srv/data/backups/` можно как staging, но **итоговая копия** должна оказаться на втором target.

### 3. Change — `/usr/local/bin/lab-backup.sh`

Требования: `set -euo pipefail`; датированный каталог; rsync/tar; манифест пакетов; лог `/var/log/lab-backup.log`; ротация N=5 на target; `--dry-run`; ненулевой exit при ошибке; после копирования — `MANIFEST.sha256`.

### 4. Verify — dry-run и прогон

Evidence: dry-run, успешный run, `ls` на **обоих** местах (staging и off-host).

### 5. Change — timer

`lab-backup.service` + `lab-backup.timer` (учебно каждые 30–60 мин; в заметках — prod nightly + offsite).  
`systemctl list-timers | grep lab-backup`.

### 6. Verify — restore файла и каталога

Классическое удаление тестового файла/каталога → restore из off-host → `sha256sum -c`. Негатив: несуществующий timestamp → явная ошибка.

### 7. Verify — timed restore drill (must)

1. Подготовьте сценарий: «пропал `/var/www/lab`».
2. Засеките время от старта восстановления до `curl -k https://127.0.0.1/` OK.
3. Запишите в `~/lab-notes/11-restore-drill.md`: wall-clock минуты/секунды, уложились ли в RTO, что тормозит.
4. Если не уложились — одна итерация улучшения (скрипт `lab-restore-www.sh` или runbook) и повтор замера.

### 8. Verify — инъекция сбоя

Несуществующий source → ненулевой код + строка в логе → откат.

### 9. Automate — report

`/usr/local/bin/lab-backup-report.sh`: список копий на off-host, размер, возраст; если старше RPO — exit 1.

### 10. Document

Политика: что / куда (два target) / частота / RPO / RTO / как проверить / ограничения (нет географического offsite — долг).

## Критерии приёмки

- [ ] Off-host target реально получает копии
- [ ] Скрипт + dry-run + ротация + MANIFEST
- [ ] Timer активен; RPO числом согласован с расписанием
- [ ] Restore файла и каталога доказаны
- [ ] Timed drill задокументирован; RTO числом
- [ ] Report свежести; негатив сбоя отработан

## Подсказки

| Семья | Манифест пакетов |
|-------|------------------|
| Debian/Ubuntu | `dpkg -l` |
| Rocky/Alma | `rpm -qa` |

SSH на `cli`: отдельный ключ, пользователь `backup` с правом записи только в `/var/backups/srv`.

## Очистка

Оставьте скрипт, timer, ≥1 off-host копию для капстоуна.
