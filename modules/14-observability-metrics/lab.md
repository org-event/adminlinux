# Лаборатория 14 — метрики и alert drill

## Цель

Собрать pull-стек на двух ВМ, правила disk/service/HTTP, провести drill: алерт загорелся → устранили → погас.

## Окружение

- `srv` + `cli`, lab-сеть, HTTPS из 10
- RAM на `cli` ≥1.5 ГБ желательно
- Firewall: разрешите scrape-порты **только** из lab (например `9100/tcp` с IP `cli`)

## Задания

### 1. Осмотрите стенд

```bash
# на srv и cli
free -h; ss -tulpn; ip -br a
```

Выберите размещение: Prometheus на `cli` (рекомендуется), node_exporter на `srv`.

### 2. Поставьте — node_exporter на `srv`

1. Установите из репо **или** official binary в `/opt/node_exporter`.
2. systemd unit, enable --now.
3. Слушать `0.0.0.0:9100` или lab IP; firewall — только `cli`.
4. Локально: `curl -s http://127.0.0.1:9100/metrics | head`.

### 3. Поднимите — Prometheus lite на `cli`

1. Установите Prometheus (пакет/binary).
2. `scrape_configs`: job `node` → `srv:9100`.
3. UI/API: `http://<cli>:9090/targets` — state **UP**.
4. Пакет доказательств: screenshot не обязателен; `curl` API targets → `14-targets.json`.

### 4. Добавьте — alert rules (минимум 3)

Файл rules + `rule_files` в prometheus.yml:

1. **DiskLow** — мало места на `/` или `/srv/data` (для drill можно временно жёсткий порог, например `< 95% free` → наоборот low avail).
2. **TargetDown** / service — `up == 0` >1m.
3. **HttpFail** — один из вариантов:
   - blackbox exporter probing `https://srv/` (с insecure skip verify в lab), или
   - учебный oneshot exporter / `curl` side-car, пишущий gauge `lab_http_up`.

`promtool check rules` (если есть) или перезагрузка без ошибок в логе.

### 5. Проведите — alert drill (обязательно)

1. Спровоцируйте **один** алерт (остановить node_exporter / заполнить учебный volume loop-файлом / сломать HTTP).
2. Дождитесь **firing** (снизьте `for:` до 30s–1m для учёбы).
3. Пакет доказательств: API `/api/v1/alerts` → `14-alert-firing.json`.
4. Устраните причину → alert inactive/resolved → `14-alert-resolved.json`.
5. Хронология в `14-alert-drill.md`. Если не вышло — проверьте `for:` и что rule file реально загружен.

### 6. Свяжите с health-check 09

В заметках: что ловит timer на srv vs что ловит Prometheus. Не удаляйте health-check — дополните ссылкой на alerts.

### 7. Автоматизируйте smoke

`/usr/local/bin/lab-metrics-smoke.sh` (на cli или srv):

- targets API содержит UP для job node;
- хотя бы одно rule file загружено;
- exit 1 при DOWN.

### 8. Задокументируйте

Топология портов; кто скрейпит кого; пороги; ограничения (нет внешнего Alertmanager/Pager — долг).

## Критерии приёмки

- [ ] node_exporter отдаёт metrics; порт не открыт «в мир»
- [ ] Prometheus видит target UP
- [ ] ≥3 alert rules (disk, service/target, HTTP)
- [ ] Alert drill: firing → resolve, JSON сохранён в заметках
- [ ] Smoke-скрипт зелёный
- [ ] Заметки связывают модуль 09 и 14

## Подсказки

| Семья | Заметки |
|-------|---------|
| Debian/Ubuntu | пакеты могут называться `prometheus`, `prometheus-node-exporter` |
| Rocky/Alma | часто удобнее binary из GitHub releases в `/opt` |

SELinux: exporter binary/port — при AVC см. модуль 13.

## Очистка

Стек оставьте для капстоуна. Учебный loop-файл диска удалите.
