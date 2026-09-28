# Лаборатория 09 — журналы и мониторинг

## Цель

Провести мини-расследование по журналам и автоматизировать health-check.

## Окружение

- `srv` после модулей 06–08

## Задания

### 1. Inspect — карта логов

```bash
journalctl --disk-usage
ls -lah /var/log | head
sudo journalctl -b -p err..alert --no-pager | tee ~/lab-notes/09-boot-errors.txt
systemctl --failed
```

### 2. Change — смоделировать событие

Перезапустите SSH-сервис и найдите соответствующие строки в journal по unit.

```bash
sudo systemctl restart ssh || sudo systemctl restart sshd
sudo journalctl -u ssh -u sshd --since "5 min ago" --no-pager
```

### 3. Change — health-check скрипт

Создайте `/usr/local/bin/lab-healthcheck.sh`, который проверяет:

- `df` не выше порога (например 90%) на `/` и `/srv/data` (если есть);
- нет failed systemd units;
- SSH порт слушает (`ss`);
- пишет результат в `/var/log/lab-healthcheck.log` и возвращает ненулевой код при аварии.

Подключите к timer (можно новый или расширьте идею модуля 06).

### 4. Verify

Запустите скрипт вручную, затем дождитесь timer. Покажите успешный и (искусственно) неуспешный сценарий — например, временно сломайте проверку пути и верните обратно.

### 5. Document

В `~/lab-notes/09.md` опишите алгоритм: «сервис не отвечает — куда смотреть за 10 минут».

## Критерии приёмки

- [ ] Есть выгрузка ошибок загрузки
- [ ] Найдены journal-события рестарта SSH
- [ ] Health-check скрипт существует и логирует
- [ ] Timer/сервис запускает проверку
- [ ] Есть короткий troubleshooting-алгоритм

## Подсказки

- Для порога диска удобен `df --output=pcent,target`.
- Не паритесь идеальным парсингом — важны ясность и код возврата.

## Очистка

Скрипт оставьте для капстоуна.
