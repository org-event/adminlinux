# Модуль 17 — Identity: SSSD + LDAP lite

**Цель:** центральный логин через OpenLDAP (или FreeIPA-lite) + SSSD: `getent`, группы/sudo, локальный admin на случай отказа каталога (break-glass).

**Время:** 4–6 часов  
**Зависит от:** 03, 08 (рекомендуется 13 для нюансов SELinux/AppArmor)  
**Планка:** рабочий стенд на 1–2 ВМ, не полный enterprise FreeIPA.

| Файл | Содержание |
|------|------------|
| [theory.md](theory.md) | LDAP vs SSSD, sudo, break-glass |
| [lab.md](lab.md) | OpenLDAP+SSSD на srv, логин с cli |
| [checklist.md](checklist.md) | Самопроверка |
