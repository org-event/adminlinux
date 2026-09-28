# Лаборатория 04 — аудит пакетов

## Цель

Поставить админ-набор, уметь ответить «что/откуда/зачем», отработать hold и безопасный remove/reinstall.

## Окружение

- `srv` с NAT/интернетом

## Задания

### 1. Inspect — backlog обновлений

| Семья | Команда |
|-------|---------|
| Debian/Ubuntu | `sudo apt update` → число upgradable |
| Rocky/Alma | `sudo dnf check-update` → оценка объёма |

Число/вывод → `04.md`. Список репо (`sources` / `yum.repos.d`) — кратко: какие enabled.

### 2. Change — набор

Установите: `tree`, `curl`, `jq`, `htop` (или `btop`), пакет для `dig`:

| Семья | dig |
|-------|-----|
| Debian/Ubuntu | `dnsutils` |
| Rocky/Alma | `bind-utils` |

### 3. Verify — паспорт `curl`

В `04-curl.txt`: версия, policy/repo, список файлов (head), `command -v`.

### 4. Verify — провайдер `dig`

Команда, пакет, `dig example.com +short` → evidence.

### 5. Change — hold / versionlock

1. Зафиксируйте `jq` (hold или versionlock).
2. Покажите статус hold/lock.
3. Снимите фиксацию.
4. Если на RHEL нет versionlock — поставьте плагин **или** опишите ограничение и покажите `dnf history` как контроль изменений.

### 6. Change — remove/reinstall `tree`

Доказать отсутствие бинаря → вернуть. В заметках: когда `autoremove` опасен.

### 7. Automate

`~/lab-notes/bin/check-admin-tools.sh`: `curl jq tree dig` + `htop|btop`. Exit 1 при MISSING.

### 8. Document

`04.md`: update vs upgrade; риск `curl|bash`; зачем hold на mid-стенде.

## Критерии приёмки

- [ ] Набор в PATH; dig работает
- [ ] Паспорт curl + провайдер dig
- [ ] Hold/lock продемонстрирован (или ограничение задокументировано)
- [ ] remove/reinstall tree
- [ ] Check-скрипт зелёный

## Подсказки

Не смешивайте сторонние репо «для красоты». Для mid+ достаточно штатных.

## Очистка

Пакеты оставьте.
