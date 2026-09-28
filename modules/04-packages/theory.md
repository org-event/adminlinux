# Теория — пакеты (mid)

## Норма vs исключение

Норма: штатный менеджер + подписанные репо.  
Исключение: сторонний репо / бинарник с сайта — только с записью риска.

## Операции

| Действие | Debian/Ubuntu | Rocky/Alma |
|----------|---------------|------------|
| Индексы | `apt update` | `dnf makecache` / check-update |
| Установка | `apt install` | `dnf install` |
| Инфо | `apt show` / `apt-cache policy` | `dnf info` |
| Файлы | `dpkg -L` | `rpm -ql` |
| Провайдер | `dpkg -S` / `apt-file` | `dnf provides` / `rpm -qf` |
| Hold/pin | `apt-mark hold` | `dnf versionlock` (плагин) |

## Hold / pin

Учебный приём: зафиксировать версию утилиты, показать, что массовый upgrade её не трогает (или как снимается hold). Без versionlock на RHEL — опишите эквивалент или поставьте плагин.
