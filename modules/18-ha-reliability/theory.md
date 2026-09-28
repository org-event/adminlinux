# Теория — HA lite и SPOF

## SPOF

Single Point of Failure — компонент, чей отказ роняет сервис целиком.  
HA lite в курсе: **убрать один** SPOF на пути клиента к HTTP, осознанно оставив другие (один диск, один LDAP, один Prometheus).

## Два рабочих пути lab

### A. keepalived + VIP

```text
cli/curl  -->  VIP (float)
               /          \
            srv-a        srv-b   (или srv + вторая роль на cli)
```

VRRP держит VIP на master; при падении — backup забирает VIP. Нужна L2-связность lab или осознанный cloud VIP (документируйте отличия).

### B. nginx upstream failover (без VIP)

```text
cli --> nginx (на cli или отдельном LB-хосте)
            upstream backend_a; backend_b;  (max_fails / fail_timeout)
```

Останавливаете один backend — трафик идёт на живой. SPOF остаётся сам LB — это честно пишется в карту.

## Что не делаем

- Full Pacemaker/Corosync cluster на 5 нод;
- распределённое хранилище;
- «магический» zero-downtime без измерений.

## Метрики отказа

Фиксируйте: время detect → failover → восстановление ответа `curl` (секунды). Связь с **14** (TargetDown) и **16** (нагрузка на оставшийся узел).
