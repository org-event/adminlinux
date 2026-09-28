# Лаборатория 13 — отказ → логи → fix

## Цель

Воспроизвести MAC-отказ на учебной службе (nginx и/или кастомный путь), снять evidence из логов и устранить **правильным** контролом; финал — MAC включён.

## Окружение

- `srv` с nginx/TLS из модуля 10
- Снимок `before-mac`
- Ветка: **ваша** семья дистрибутива (ниже — обе)

## Задания

### 1. Inspect — baseline

**Rocky/Alma:**

```bash
getenforce
sestatus
sudo ausearch -m avc -ts today 2>/dev/null | tail || true
```

**Debian/Ubuntu:**

```bash
sudo aa-status
sudo journalctl -b --no-pager | grep -i apparmor | tail -30 || true
```

Сохраните baseline в `~/lab-notes/13-baseline.txt`. В `13.md` — одна страница: чем SELinux отличается от AppArmor на уровне модели.

### 2. Change — спровоцировать отказ

Выберите **один** сценарий (или эквивалент с тем же циклом):

| # | Идея |
|---|------|
| A | Document root / TLS key в нестандартном пути без контекста/профиля |
| B | Nginx/httpd должен сделать network connect (proxy) — boolean закрыт |
| C | Кастомный unit пишет в `/srv/...` — deny |

Зафиксируйте симптомы: HTTP 403/500, journal ошибки, клиентский curl.

**Запрещено** как финал: `SELINUX=disabled`, `setenforce 0` оставить, purge AppArmor.

### 3. Verify — снять AVC / AppArmor DENIED

Evidence в `13-avc.txt` / `13-apparmor-deny.txt`: timestamp, процесс, denied operation, путь/порт.

Краткий Permissive/complain **допустим** на время сбора (запишите `date -Is` вкл/выкл), затем верните enforce.

### 4. Change — fix

| SELinux | AppArmor |
|---------|----------|
| `restorecon` / `semanage fcontext` | правка профиля / `aa-logprof` |
| `setsebool -P …` | tune профиля, не disable |
| перезапуск службы | `apparmor_parser -r`, restart |

Покажите: сервис работает **и** MAC снова enforcing/enabled.

### 5. Verify — негатив регрессии

Повторно сломайте контекст/профиль осознанно (или уберие boolean) → снова deny → снова fix. Второй цикл короче, но evidence обязателен.

### 6. Automate

`/usr/local/bin/lab-mac-status.sh`:

- SELinux: печатает getenforce; exit 1 если не Enforcing
- AppArmor: `aa-status --enabled` / нет «dead»; exit 1 если подсистема выключена
- Опционально: `curl -sk -o /dev / https://127.0.0.1/` 

### 7. Document

Runbook в `13.md`: симптомы → какие логи → допустимые фиксы → что запрещено. Ссылка из handoff капстоуна.

## Критерии приёмки

- [ ] Baseline MAC снят
- [ ] Отказ воспроизведён и залогирован (AVC/DENIED)
- [ ] Fix без постоянного disable
- [ ] Финал: служба OK + MAC включён
- [ ] Повторный цикл deny→fix показан
- [ ] `lab-mac-status.sh` зелёный

## Подсказки

| | Rocky/Alma | Debian/Ubuntu |
|--|------------|---------------|
| Статус | `getenforce` | `aa-status` |
| Пакеты помощи | `setroubleshoot`, `policycoreutils-python-utils` | `apparmor-utils` |
| Частый nginx | `httpd_sys_content_t`, booleans | профиль `/etc/apparmor.d/nginx` или synlinks |

## Очистка

Оставьте MAC во включённом режиме. Снимок `after-mac-ok`.
