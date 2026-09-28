# Лаборатория 04 — пакеты и обновления

## Цель

Поставить админ-утилиты, уметь отвечать «что за пакет и откуда», найти провайдера команды и безопасно проверить удаление/поиск.

## Окружение

- ВМ `srv` с доступом в интернет (NAT)

## Задания

### 1. Inspect

```bash
cat /etc/os-release

# Debian/Ubuntu
sudo apt update
apt list --upgradable 2>/dev/null | head
apt list --upgradable 2>/dev/null | wc -l

# Rocky/Alma
sudo dnf check-update | head
sudo dnf repolist
```

Запишите примерно число доступных обновлений в `~/lab-notes/04.md`.

### 2. Change — установка набора

Установите: `tree`, `curl`, `jq`, `htop` (или `btop`, если есть в репозитории).

Проверьте: `command -v` для каждой утилиты.

### 3. Verify — паспорт пакета `curl`

В `~/lab-notes/04-curl.txt`:

| Семья | Описание / файлы / версия |
|-------|---------------------------|
| Debian/Ubuntu | `apt show curl`, `dpkg -L curl \| head`, `apt-cache policy curl` |
| Rocky/Alma | `dnf info curl`, `rpm -ql curl \| head`, `dnf list installed curl` |

Плюс `command -v curl` и `curl --version | head -1`.

### 4. Change — поиск провайдера команды `dig`

| Семья | Пакет | Установка |
|-------|-------|-----------|
| Debian/Ubuntu | `dnsutils` | `sudo apt install dnsutils` |
| Rocky/Alma | `bind-utils` | `sudo dnf install bind-utils` |

Поиск «кто даёт бинарь»:

```bash
# Debian: apt-file может отсутствовать — альтернатива:
dpkg -S "$(command -v dig)" 2>/dev/null || true
# RHEL:
rpm -qf "$(command -v dig)"
dnf provides '*/dig' | head
```

Проверка: `dig example.com +short`.

### 5. Verify — история и зависимости

1. Покажите недавно установленные пакеты (как умеете: `/var/log/apt/history.log`, `dnf history`, или дата файлов).
2. Для `jq` покажите зависимости (`apt depends jq` / `dnf deplist jq` или `rpm -qR`).
3. В заметках: чем «рекомендует» пакет отличается от жёсткой зависимости (своими словами).

### 6. Change — удаление и возврат (безопасный пакет)

1. Удалите `tree` (`apt remove` / `dnf remove`).
2. Убедитесь, что `command -v tree` пуст.
3. Поставьте `tree` обратно.
4. **Не** используйте `autoremove` вслепую на всём стенде; опишите в заметках, когда он нужен.

### 7. Automate — список админ-утилит

Скрипт `~/lab-notes/bin/check-admin-tools.sh` проверяет наличие `curl jq tree dig htop|btop` и печатает OK/MISSING. Exit 1 если чего-то нет.

### 8. Document

В `~/lab-notes/04.md`:

- Чем `apt update` отличается от `apt upgrade`? (для dnf — `check-update` vs `upgrade`)
- Почему опасны случайные RPM/DEB из чатов?
- Как откатить мысль «поставлю из curl \| bash»?

## Критерии приёмки

- [ ] Утилиты установлены и в PATH
- [ ] Есть паспорт пакета `curl`
- [ ] `dig` работает; найден пакет-провайдер
- [ ] Показан цикл remove/reinstall для `tree`
- [ ] Есть check-скрипт и ответы Document

## Подсказки

| Команда | Debian | RHEL |
|---------|--------|------|
| обновить индексы | `apt update` | метаданные тянет `dnf` |
| пакет для dig | `dnsutils` | `bind-utils` |
| файлы пакета | `dpkg -L` | `rpm -ql` |

## Очистка

Пакеты оставьте — пригодятся дальше.
