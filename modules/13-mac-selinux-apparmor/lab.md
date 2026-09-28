# Лаборатория 13 — отказ → логи → fix

## Цель

Воспроизвести MAC-отказ на учебной службе (nginx и/или кастомный путь), снять пакет доказательств из логов и устранить **правильным** контролом. Финал — MAC включён.

## Окружение

- `srv` с nginx/TLS из модуля 10
- Снимок `before-mac`
- Ветка: **ваша** семья дистрибутива (ниже — обе)

## Задания

### 1. Снимите baseline

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

### 2. Спровоцируйте отказ

Выберите **один** сценарий (или эквивалент с тем же циклом):

| # | Идея |
|---|------|
| A | Document root / TLS key в нестандартном пути без контекста/профиля |
| B | Nginx/httpd должен сделать network connect (proxy) — boolean закрыт |
| C | Кастомный unit пишет в `/srv/...` — deny |

Зафиксируйте симптомы: HTTP 403/500, journal ошибки, клиентский curl.

**Запрещено** как финал: `SELINUX=disabled`, оставить `setenforce 0`, purge AppArmor. Не отключайте SELinux «навсегда».

### 3. Снимите AVC / AppArmor DENIED

Пакет доказательств в `13-avc.txt` / `13-apparmor-deny.txt`: timestamp, процесс, denied operation, путь/порт.

Краткий Permissive/complain **допустим** на время сбора (запишите `date -Is` вкл/выкл). Затем верните enforce.

### 4. Исправьте правильно

| SELinux | AppArmor |
|---------|----------|
| `restorecon` / `semanage fcontext` | правка профиля / `aa-logprof` |
| `setsebool -P …` | tune профиля, не disable |
| перезапуск службы | `apparmor_parser -r`, restart |

Проверьте: сервис работает **и** MAC снова enforcing/enabled. Если не вышло — перечитайте AVC: часто хватает `restorecon` или одного boolean.

### 5. Повторите цикл регрессии

Снова сломайте контекст/профиль осознанно (или уберите boolean) → снова deny → снова fix. Второй цикл короче, но пакет доказательств обязателен.

### 6. Автоматизируйте статус

`/usr/local/bin/lab-mac-status.sh`:

- SELinux: печатает getenforce; exit 1 если не Enforcing
- AppArmor: `aa-status --enabled` / нет «dead»; exit 1 если подсистема выключена
- Опционально: `curl -sk -o /dev/null https://127.0.0.1/` 

### 7. Задокументируйте

Runbook в `13.md`: симптомы → какие логи → допустимые фиксы → что запрещено. Ссылка из документа передачи смены (`HANDOFF.md`) капстоуна.

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
| Частый nginx | `httpd_sys_content_t`, booleans | профиль `/etc/apparmor.d/nginx` или symlinks |

## Очистка

Оставьте MAC во включённом режиме. Снимок `after-mac-ok`.
