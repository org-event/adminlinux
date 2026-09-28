# Лаборатория 16 — baseline, деградация, заметки по ёмкости

## Цель

Снимите baseline `srv`, смоделируйте одну деградацию, найдите виновника через vmstat/iostat/pidstat и оформите пороги плюс заметки по ёмкости.

## Окружение

- `srv` (+ `cli` по желанию для генерации HTTP-нагрузки)
- Пакет `sysstat` / `stress-ng` (или эквивалент)
- Снимок гипервизора перед memory/disk stress
- Метрики модуля 14 желательны для сопоставления

## Задания

### 1. Осмотр

```bash
uptime; free -h; nproc
vmstat 1 5
# при наличии:
iostat -xz 1 3
pidstat -urd 1 3
```

Зафиксируйте idle-снимок → `~/lab-notes/16-baseline.txt` (команды + вывод).

### 2. Изменение — nominal load

С `cli` или локально: умеренный HTTP к HTTPS/nginx (модуль 10) **или** короткий бэкап-скрипт.

Сохраните `16-nominal.txt` (те же команды, что в baseline).

### 3. Изменение — сценарий деградации (один на выбор)

| Сценарий | Идея | Ожидаемый след |
|----------|------|----------------|
| A CPU | `stress-ng --cpu $(nproc) --timeout 60s` | load↑, %user↑, pidstat top = stress |
| B Disk | запись на учебный LV/loop (`dd if=/dev/zero of=/srv/data/lab-stress.bin bs=1M count=512 oflag=direct`) | iowait/util↑ |
| C Memory | `stress-ng --vm 1 --vm-bytes 70% --timeout 45s` (осторожно) | free↓, возможен swap |

Параллельно во втором SSH: `vmstat 1`, `iostat -xz 1`, `pidstat …`.

Сохраните выводы: `16-degrade-<scenario>.txt` + короткий `16-triage.md` (что увидели первым).

### 4. Проверка

1. Назовите bottleneck одной фразой (CPU / disk / memory).
2. Негатив: после остановки stress показатели возвращаются к baseline (±шум) — зафиксируйте.
3. Если есть Prometheus: отметьте, отразился ли пик на графике/метриках (ссылка на 14).

### 5. Документ — пороги и ёмкость

`16-thresholds.md`:

- load / 5m — warning и critical *для этого стенда*;
- disk util или await;
- MemAvailable / swap activity;
- что **не** алертить (шум).

`16-capacity-notes.md`: 3–5 предложений — при росте нагрузки что упрётся первым и какой сигнал смотреть.

### 6. Автоматизация

`/usr/local/bin/lab-perf-snapshot.sh` → пишет timestamp + `uptime` + `vmstat 1 3` + `free -h` в `~/lab-notes/perf/`.  
Запуск вручную или timer раз в час (включать timer не обязательно).

## Критерии приёмки

- [ ] Baseline и nominal сохранены
- [ ] Один сценарий деградации с triage-заметкой
- [ ] Bottleneck назван и подтверждён выводом инструментов
- [ ] Пороги + заметки по ёмкости написаны
- [ ] Snapshot-скрипт работает
- [ ] Цикл Осмотр→Изменение→Проверка→Документ→Автоматизация пройден

## Подсказки

| Семья | Заметки |
|-------|---------|
| Debian/Ubuntu | `apt install sysstat stress-ng` |
| Rocky/Alma | `dnf install sysstat stress-ng` |

`iostat` может требовать `systemctl enable --now sysstat` на некоторых дистрах — для lab достаточно ручных выборок.

## Очистка

Удалите `lab-stress.bin` и остановите stress. Скрипт snapshot оставьте.
