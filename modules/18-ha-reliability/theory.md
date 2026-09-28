# Теория — HA lite и SPOF

## SPOF

Single Point of Failure — узел, чей отказ роняет сервис целиком.  
В курсе HA lite значит: **убрать один** SPOF на пути клиента к HTTP и честно оставить остальные (один диск, один LDAP, один Prometheus).

## Два рабочих пути в lab

### A. keepalived + VIP

```text
cli/curl  -->  VIP (float)
               /          \
            srv-a        srv-b   (или srv + вторая роль на cli)
```

VRRP держит VIP на master; при падении backup забирает VIP. Нужна L2-связность в lab или осознанный cloud VIP — отличия запишите в заметках.

### B. nginx upstream failover (без VIP)

```text
cli --> nginx (на cli или отдельном LB-хосте)
            upstream backend_a; backend_b;  (max_fails / fail_timeout)
```

Остановите один backend — трафик уходит на живой. SPOF остаётся сам LB — так и пишите в карту.

## Чего не делаем

- полный Pacemaker/Corosync на 5 нод;
- распределённое хранилище;
- «магический» zero-downtime без замеров.

## Метрики отказа

Фиксируйте: время detect → failover → восстановление ответа `curl` (в секундах). Связь с модулем **14** (TargetDown) и **16** (нагрузка на оставшийся узел).
