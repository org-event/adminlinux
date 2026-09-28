# Модуль 18 — HA и надёжность (lite)

**Цель:** убрать один явный SPOF на пути к сервису: VIP/keepalived **или** nginx upstream failover; отказ узла с документированием оставшихся SPOF.

**Время:** 3–5 часов  
**Зависит от:** 07, 10  
**Планка:** 2 ВМ (+ по желанию VIP на одной доп. роли), не Pacemaker на 5 нод.

| Файл | Содержание |
|------|------------|
| [theory.md](theory.md) | SPOF, VIP, upstream failover |
| [lab.md](lab.md) | Failover drill + карта SPOF |
| [checklist.md](checklist.md) | Самопроверка |
