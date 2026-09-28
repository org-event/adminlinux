# Лаборатория 04 — аудит пакетов

## Цель

Поставьте админ-набор, ответьте «что / откуда / зачем», отработайте hold и безопасный remove/reinstall.

## Окружение

- `srv` с NAT и интернетом

## Задания

### 1. Осмотр — backlog обновлений

| Семья | Команда |
|-------|---------|
| Debian/Ubuntu | `sudo apt update` → число upgradable |
| Rocky/Alma | `sudo dnf check-update` → оценка объёма |

Число и вывод → `04.md`. Список репо (`sources` / `yum.repos.d`) — кратко: какие enabled.

### 2. Изменение — набор

Установите: `tree`, `curl`, `jq`, `htop` (или `btop`), пакет для `dig`:

| Семья | dig |
|-------|-----|
| Debian/Ubuntu | `dnsutils` |
| Rocky/Alma | `bind-utils` |

### 3. Проверка — паспорт `curl`

В `04-curl.txt`: версия, policy/repo, список файлов (head), `command -v`.

### 4. Проверка — провайдер `dig`

Команда, пакет, `dig example.com +short` — сохраните вывод в `~/lab-notes/`.

### 5. Изменение — hold / versionlock

1. Зафиксируйте `jq` (hold или versionlock).
2. Покажите статус hold/lock.
3. Снимите фиксацию.
4. Если на RHEL нет versionlock — поставьте плагин **или** опишите ограничение и покажите `dnf history` как контроль изменений.

### 6. Изменение — remove/reinstall `tree`

Докажите отсутствие бинаря, затем верните. В заметках: когда `autoremove` опасен.

### 7. Автоматизация

`~/lab-notes/bin/check-admin-tools.sh`: `curl jq tree dig` + `htop|btop`. Exit 1 при MISSING.

### 8. Запись

`04.md`: update vs upgrade; риск `curl|bash`; зачем hold на mid-стенде.

## Критерии приёмки

- [ ] Набор в PATH; dig работает
- [ ] Паспорт curl + провайдер dig
- [ ] Hold/lock показан (или ограничение задокументировано)
- [ ] remove/reinstall tree
- [ ] Check-скрипт зелёный

## Подсказки

Не тащите сторонние репо «для красоты». На mid+ хватает штатных. Если не вышло с сетью — вернитесь к NAT/DNS из модуля 01.

## Очистка

Пакеты оставьте.
