# Лаборатория 01 — инвентарь стенда

## Цель

Поднять `srv`, сдать операторский паспорт и inventory без двусмысленности «на какой машине я».

## Окружение

- ВМ по [lab-setup](../../course/lab-setup.md)
- `~/lab-notes/`, снимок `clean-base` в конце

## Задания

### 1. Inspect — паспорт

```bash
mkdir -p ~/lab-notes/bin
{
  echo "## passport srv"; date -Is
  cat /etc/os-release; hostnamectl; uname -r
  free -h; df -hT; lsblk; ip -br a; id
  systemctl is-system-running || true
} | tee ~/lab-notes/01-passport.txt
```

### 2. Change — идентичность

1. Hostname `srv.lab.local`.
2. `~/lab-notes/inventory.md`: hostname, IP (NAT + lab), семья дистрибутива, пользователь, hypervisor, дата снимка, план второй ВМ `cli`.

### 3. Verify — dual-distro ментальная модель

В `~/lab-notes/01.md` таблица (5–7 строк): пакеты / firewall / MAC / типичный лог-путь / «как обновить систему» — для **вашей** семьи и кратко для **другой**.

Проверка пакетного менеджера (без лишних установок):

| Семья | Команда |
|-------|---------|
| Debian/Ubuntu | `sudo apt update` |
| Rocky/Alma | `sudo dnf repolist` |

### 4. Verify — негатив «не тот хост»

Чеклист из ≥4 пунктов в `01.md` (hostname, IP/интерфейсы, отсутствие личных данных в `/home`, снимок/метка ВМ). Выполните чеклист и отметьте результат.

### 5. Automate

`~/lab-notes/bin/collect-passport.sh` — пересобирает `01-passport.txt` (timestamp обновляется). Два запуска подряд — в evidence.

### 6. Document

В `01.md`: что войдёт в `clean-base`; зачем вторая ВМ в mid+ (проверка снаружи, DR-target, metrics).

## Критерии приёмки

- [ ] Паспорт с os-release, RAM, дисками, IP, systemd state
- [ ] Hostname `srv.lab.local`, актуальный `inventory.md`
- [ ] Dual-distro таблица заполнена
- [ ] Негатив «не тот хост» выполнен
- [ ] Скрипт паспорта + снимок `clean-base`

## Подсказки

- IP: `ip -br a`. Не полагайтесь на `ifconfig`.
- Если нет интернета — NAT/DNS чините до модуля 04.

## Очистка

Откатывать нечего. Сохраните снимок.
