# Теория — метрики и алерты

## Модель

```text
srv  --:9100-->  node_exporter (метрики хоста)
cli  --pull-->   Prometheus  --rules-->  Alertmanager (опционально) / UI firing
```

Учебный минимум mid+: **Prometheus** на `cli` (или на `srv`, если RAM мало — зафиксируйте), **node_exporter** на `srv`, scraping ≥1 target UP, ≥3 alert rules, один drill до состояния firing → resolve.

## Альтернатива (если пакеты неудобны)

Допустим упрощённый, но **не bash-only** стек, например:

- VictoriaMetrics / Grafana Agent single-binary, или
- `prometheus` + `node_exporter` из release tarball в `/opt`

Не принимается как потолок: только `df` в cron без time-series и без alert rule.

## Типы алертов курса

| Алерт | Идея выражения |
|-------|----------------|
| Disk | `node_filesystem_avail_bytes` низкий на `/` или `/srv/data` |
| Service | `up{job="node"} == 0` или blackbox/exporter down; либо process absent |
| HTTP | `probe_success` (blackbox) **или** scripted exporter / nginx stub + правило; допустим side-car `curl` exporter на `:9115` учебный |

## Связь с 09

journald/health-check остаются для локального runbook. Капстоун требует именно пакет доказательств по metrics+alerts.
