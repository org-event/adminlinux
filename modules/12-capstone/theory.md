# Теория — капстоун mid+

Капстоун собирает освоенное и добавляет одно осознанное усиление из эшелона:

- SSH hardening, пользователи, LVM, firewall;
- **HTTPS** (не optional);
- NFS или обоснованная замена;
- **metrics + alerts** (модуль 14);
- стенд **поднят/сведён Ansible** (модуль 15) или playbooks apply + drift-заметки;
- **off-host DR** + timed drill (модуль 11);
- MAC в enforcing/enabled с понятным fix-путём (модуль 13);
- handoff + **postmortem** мини-инцидента;
- **эшелон (выбор):** HA ([18](../18-ha-reliability/)) **или** identity ([17](../17-identity-sssd/)) **или** containers/k8s lite ([19](../19-containers-orchestration/)); perf ([16](../16-performance-capacity/)) и advanced net ([20](../20-advanced-networking/)) — сильный should.

## Handoff

1. Зачем сервер и топология (`srv`/`cli`)  
2. Как войти (в т.ч. break-glass, если есть LDAP)  
3. Критичные службы и health/metrics  
4. Где данные и TLS-материалы  
5. Firewall / зоны / VPN (если 20)  
6. Бэкап/RPO/RTO и restore  
7. Потеря SSH  
8. HA path или контейнерный endpoint (если выбрано)  
9. Известные долги и SPOF  

## Postmortem (обязателен)

Короткий разбор: trigger → impact → detection (лог/алерт) → mitigation → root cause → action items. Без «больше не ошибаться» — только конкретные изменения.
