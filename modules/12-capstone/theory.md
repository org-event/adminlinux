# Теория — капстоун mid+

Капстоун собирает освоенное, а не вводит новый стек:

- SSH hardening, пользователи, LVM, firewall;
- **HTTPS** (не optional);
- NFS или обоснованная замена;
- **metrics + alerts** (модуль 14);
- стенд **поднят/сведён Ansible** (модуль 15) или playbooks apply + drift-заметки;
- **off-host DR** + timed drill (модуль 11);
- MAC в enforcing/enabled с понятным fix-путём (модуль 13);
- handoff + **postmortem** мини-инцидента.

## Handoff

1. Зачем сервер и топология (`srv`/`cli`)  
2. Как войти  
3. Критичные службы и health/metrics  
4. Где данные и TLS-материалы  
5. Firewall  
6. Бэкап/RPO/RTO и restore  
7. Потеря SSH  
8. Известные долги  

## Postmortem (обязателен)

Короткий разбор: trigger → impact → detection (лог/алерт) → mitigation → root cause → action items. Без «больше не ошибаться» — только конкретные изменения.
