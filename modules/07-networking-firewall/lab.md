# Лаборатория 07 — сеть и firewall

## Что сделаете

Соберёте сетевой паспорт, разделите сбои IP/DNS/маршрута, включите firewall с разрешённым SSH и подготовите HTTP к модулю 10.

## Что понадобится

- `srv` (+ настоятельно рекомендуется `cli`)
- Снимок `before-firewall`
- Доступ к **консоли гипервизора** (на случай блокировки SSH)

## Ход работы

### 1. Осмотр — сетевой паспорт

Сохраните вывод в `~/lab-notes/07-net.txt`:

```bash
{
  echo "=== $(date -Is) ==="
  ip -br a
  ip -4 addr
  ip r
  ip -6 r || true
  cat /etc/resolv.conf
  command -v resolvectl >/dev/null && resolvectl status || true
  ss -tulpn
  ping -c 2 1.1.1.1
  ping -c 2 example.com
} | tee ~/lab-notes/07-net.txt
```

Отметьте в заметках: какой интерфейс lab/host-only, какой NAT.

### 2. Изменение — DNS-инцидент (изолированно)

Сломайте временно DNS **только в лаборатории**:

1. Сохраните копию `resolv.conf` / зафиксируйте `resolvectl`.
2. Закомментируйте nameserver **или** покажите сбой через `resolvectl` (если stub перезаписывает файл).
3. Симптом: `ping` по IP работает, по имени — нет. Сохраните выводы в `~/lab-notes/07-dns-incident.txt`.
4. Верните DNS и подтвердите `ping example.com`.

### 3. Изменение — маршрут vs link

1. Найдите default gateway: `ip r | grep default`.
2. (Осторожно) кратко опишите в заметках, что будет, если gateway недоступен: какие проверки сначала (`ip l`, `ip r`, ping gw, ping внешнего IP, DNS).
3. С `cli` или с `srv`: `traceroute`/`tracepath` до внешнего IP (если пакет установлен) — сохраните 5–10 строк.

| Семья | Пакет traceroute |
|-------|------------------|
| Debian/Ubuntu | `traceroute` или `iputils-tracepath` |
| Rocky/Alma | `traceroute` / `tracepath` |

### 4. Изменение — firewall с безопасным порядком

1. **До** enable: явно разрешите SSH (`22/tcp` или сервис `ssh`/`sshd`).
2. Разрешите `80/tcp` заранее к модулю 10.
3. Включите firewall.
4. Проверьте, что текущая SSH-сессия жива; откройте **второе** окно и зайдите снова. Оставьте запасную сессию, пока не убедитесь.

Debian/Ubuntu (`ufw` — типичный учебный путь):

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status verbose
```

Rocky/Alma (`firewalld`):

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

Если не вышло зайти повторно — консоль гипервизора, ослабьте правила, проверьте `ss`.

### 5. Проверка с `cli` — порты и различия

С `cli`:

```bash
ping -c 2 <IP-srv>
nc -vz <IP-srv> 22
nc -vz <IP-srv> 80
nc -vz <IP-srv> 443   # ожидаемо закрыто/отфильтровано — зафиксируйте
```

В сохранённом выводе отметьте разницу:

- firewall **отбросил** vs
- firewall пропустил, но **нет listener** (`ss` пусто на :80).

### 6. Проверка — мышление default deny

1. Выведите текущие правила (ufw/firewalld).
2. Убедитесь, что **лишние** порты не открыты «на всякий случай».
3. В заметках перечислите каждое разрешённое правило и зачем оно.

### 7. Автоматизация — снимок сетевого состояния

Скрипт `/usr/local/bin/lab-net-snapshot.sh` пишет timestamped файл в `~/lab-notes/net-snaps/` с `ip -br a`, `ip r`, `ss -tulpn`, статусом firewall. Запустите дважды и сравните (`diff`).

### 8. Запись

В `~/lab-notes/07.md` — короткий runbook: «потеряли SSH после firewall — что делать» (консоль → ослабить правила → проверка `ss` → новая SSH-сессия).

## Когда считать сданным

- [ ] Есть сетевой паспорт
- [ ] DNS-инцидент воспроизведён и откатан
- [ ] Firewall включён, SSH доступен (повторный логин)
- [ ] Порт 80 разрешён правилом (к модулю 10)
- [ ] С `cli` проверены 22/80/443, различия зафиксированы
- [ ] Есть скрипт net-snapshot
- [ ] Есть runbook восстановления

## На заметку

- На облачных ВМ часто есть security group **поверх** OS-firewall.
- `ss -tulpn` — listeners; firewall — другая ось.
- Не тестируйте firewall только с localhost — localhost часто не режет так же, как внешний вход.

## После лаборатории

Firewall оставьте включённым — это целевое состояние. Не отключайте firewall «навсегда».
