# Лаборатория 09 — журналы и мониторинг

## Цель

Провести мини-расследование по журналам, автоматизировать health-check и отработать ложный/реальный FAIL.

## Окружение

- `srv` после модулей 06–08
- Желательно nginx ещё не обязателен; если уже есть (модуль 10) — используйте его в корреляции

## Задания

### 1. Inspect — карта логов и места

```bash
journalctl --disk-usage
ls -lah /var/log | head -30
sudo du -sh /var/log/* 2>/dev/null | sort -h | tail -15
sudo journalctl -b -p err..alert --no-pager | tee ~/lab-notes/09-boot-errors.txt
systemctl --failed --no-pager
```

В `~/lab-notes/09.md` кратко: куда смотреть сначала — journal или файлы в `/var/log` — и почему на вашем дистрибутиве.

### 2. Change — смоделировать событие и найти его

```bash
sudo systemctl restart ssh || sudo systemctl restart sshd
sudo journalctl -u ssh -u sshd --since "5 min ago" --no-pager | tee ~/lab-notes/09-ssh-restart.txt
```

Дополнительно:

1. Сделайте неудачный SSH-логин с `cli` (неверный пользователь/ключ).
2. Найдите событие в journal (`sshd`/`ssh`) и сохраните 5–10 релевантных строк.

### 3. Change — фильтры journalctl (тренировка)

Сохраните в `~/lab-notes/09-filters.txt` результаты:

```bash
# по приоритету
sudo journalctl -p warning -n 20 --no-pager
# за последний час
sudo journalctl --since "1 hour ago" -p err --no-pager | head
# follow на 10 секунд (Ctrl+C) — опишите в заметках, когда это нужно
# sudo journalctl -f -u ssh -u sshd
```

### 4. Change — health-check скрипт

Создайте `/usr/local/bin/lab-healthcheck.sh`, который проверяет:

- `df` не выше порога (например 90%) на `/` и `/srv/data` (если смонтирован);
- нет failed systemd units (`systemctl --failed`);
- SSH-порт слушает (`ss -ltn | grep ':22'`);
- пишет timestamped результат в `/var/log/lab-healthcheck.log`;
- код возврата **0** при OK, **ненулевой** при аварии.

Подключите timer (новый `lab-healthcheck.timer` или расширьте паттерн модуля 06), интервал 2–5 минут.

### 5. Verify — успешный и неуспешный прогон

1. Запустите скрипт вручную — ожидайте OK.
2. **Искусственно** сломайте одну проверку (например, временно укажите несуществующий mountpoint в скрипте или создайте failed unit из модуля 06).
3. Зафиксируйте FAIL в логе и код возврата.
4. Верните проверку в норму, подтвердите OK.
5. Дождитесь срабатывания timer.

Evidence: `~/lab-notes/09-health-ok.txt` и `~/lab-notes/09-health-fail.txt`.

### 6. Change — корреляция (мини-инцидент)

Сценарий:

1. Перезапустите SSH.
2. (Если nginx ещё нет — пропустите HTTP-часть или поставьте пакет заранее.)
3. В течение 5 минут соберите «таймлайн» в `~/lab-notes/09-timeline.md`: что менялось, какие unit, какие ошибки.

Цель — привычка смотреть **время** событий, а не только «есть error».

### 7. Inspect — logrotate / вакуум journal (осознанно)

1. Посмотрите примеры: `/etc/logrotate.conf`, `/etc/logrotate.d/` (не ломая).
2. Узнайте политику journal: `journalctl --disk-usage`, при наличии — `SystemMaxUse` в `/etc/systemd/journald.conf` (только читайте или меняйте на учебном стенде с записью в заметках).
3. **Не** делайте агрессивный `vacuum` на проде; на стенде допустим `journalctl --vacuum-size=50M` с фиксацией до/после disk-usage.

### 8. Automate — ежедневный дамп failed units

Скрипт или oneshot, который раз в сутки (или вручную) пишет `systemctl --failed` в `~/lab-notes/failed-$(date +%F).txt`. Покажите один ручной запуск.

### 9. Document

В `~/lab-notes/09.md` — алгоритм «сервис не отвечает — куда смотреть за 10 минут» (process/listen → systemctl → journal → disk → firewall).

## Критерии приёмки

- [ ] Есть выгрузка ошибок загрузки
- [ ] Найдены journal-события рестарта SSH (+ неудачный логин)
- [ ] Health-check скрипт логирует и возвращает корректный код
- [ ] Показаны OK и искусственный FAIL
- [ ] Timer запускает проверку
- [ ] Есть timeline мини-инцидента и troubleshooting-алгоритм

## Подсказки

- Для порога диска удобен `df --output=pcent,target`.
- Код возврата важнее «красивого» парсинга.
- Имя unit SSH: `ssh` vs `sshd`.

## Очистка

Скрипты и timer оставьте для капстоуна.
