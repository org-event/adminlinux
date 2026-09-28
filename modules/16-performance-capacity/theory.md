# Теория — performance и capacity

## Load vs pressure

| Сигнал | Что означает | Ограничение |
|--------|--------------|-------------|
| `load average` | очередь runnable+uninterruptible | не говорит *кто* и *почему* |
| CPU % | занятость ядер | высокий CPU ≠ always bad |
| iowait / disk util | диск/storage bottleneck | смотрите очередь и latency |
| PSI (pressure stall, если есть) | доля времени в stall | удобнее load на контейнерах |
| RAM / swap | memory pressure | swap activity важнее «free» |

Правило mid+: **сначала Inspect фактами**, потом гипотеза. Не крутите `nice` наугад.

## Инструменты курса

Минимум на обеих семьях (пакет `sysstat` или аналог):

- `vmstat 1` — CPU, memory, swap, io, system
- `iostat -xz 1` — диски util/await/aqu-sz
- `pidstat -urd 1` — кто жрёт CPU/IO/память
- `uptime`, `free -h`, `ss -s` — быстрый контекст

Дополнительно (не must): `sar`, `perf top` (обзорно), `systemd-cgtop`.

Связь с модулем **14**: time-series ловит тренд; этот модуль учит **живой triage** и capacity notes.

## Baseline и пороги

Baseline — снимок «норма» на вашем стенде (не чужие цифры из блога):

1. idle 2–3 минуты → `16-baseline.txt`;
2. типичная нагрузка (nginx curl loop / бэкап) → `16-nominal.txt`;
3. пороги для алертов/заметок: load, disk util, avail RAM, iowait.

Capacity note отвечает: при каком росте пользователей/RPS/размера данных стенд упрётся первым (CPU / disk / RAM / сеть).

## Деградация (учебная)

Безопасные сценарии на ВМ:

- CPU: `stress-ng --cpu N` или цикл сжатия;
- disk: `dd`/`fio` на учебный volume (не на корень прод-данных);
- memory: controlled stress с лимитом и готовностью к OOM на ВМ.

После сценария — Verify: какой инструмент указал виновника первым.
