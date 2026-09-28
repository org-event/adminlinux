# Теория — пакеты и обновления

## Зачем пакетный менеджер

Он решает зависимости, цифровые подписи репозиториев и единый путь обновлений. Установка «скачал tar.gz с сайта» — исключение, не норма.

## Debian / Ubuntu

```bash
sudo apt update
apt list --upgradable
sudo apt install -y tree curl jq
apt show curl
dpkg -L curl | head
apt rdepends curl
sudo apt remove -y package
sudo apt autoremove -y
```

Источники: `/etc/apt/sources.list`, `/etc/apt/sources.list.d/`.

## RHEL / Rocky / Alma

```bash
sudo dnf check-update
sudo dnf install -y tree curl jq
dnf info curl
rpm -ql curl | head
dnf repoquery --whatrequires curl
sudo dnf remove -y package
```

Репозитории: `/etc/yum.repos.d/`.

## Хорошие практики

- Обновляйте тестовый стенд перед продом.
- Фиксируйте в заметках, *зачем* пакет установлен.
- Не смешивайте сторонние репозитории без необходимости.
- После массового обновления перезагрузите, если обновилось ядро.
