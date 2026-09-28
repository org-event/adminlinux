# Лаборатория 20 — DNS, WireGuard, firewall policy

## Цель

Поднять локальный DNS для имён lab, туннель WireGuard srv↔cli и явную зонную/egress-политику с негативными тестами.

## Окружение

- `srv` + `cli`, lab-сеть
- Снимок `before-advnet`
- Не ломайте единственный SSH path до проверки WG+console

## Задания

### 1. Inspect

```bash
resolvectl status 2>/dev/null || cat /etc/resolv.conf
ip -br a; sudo nft list ruleset 2>/dev/null | head || firewall-cmd --get-active-zones
```

Зафиксируйте текущий DNS и открытые сервисы → `20-before.txt`.

### 2. Change — DNS-as-service на `srv`

1. Установите **dnsmasq** или **Unbound**.
2. Статические имена: `srv.lab` → IP srv, `cli.lab` → IP cli (и при наличии VIP — `app.lab`).
3. Слушать lab IP / localhost; firewall: UDP/TCP 53 только из lab.
4. На `cli` (и srv): nameserver = IP srv (netplan/NetworkManager/`resolv.conf` — по дистрибутиву, с бэкапом).

Verify:

```bash
dig +short srv.lab @<IP-srv>
ping -c 1 srv.lab
```

Негатив: запрос с «чужого» адреса (если можете) или отсутствие записи → NXDOMAIN.

Evidence: `20-dns.txt`.

### 3. Change — WireGuard srv↔cli

1. Пакет `wireguard` / `wireguard-tools`.
2. Ключи на обоих хостах; конфиги `/etc/wireguard/wg0.conf` (PrivateKey, Address, Peer PublicKey, Endpoint=lab IP, AllowedIPs=10.66.0.0/24, PersistentKeepalive=25 на cli).
3. `wg-quick up wg0` / enable `wg-quick@wg0`.
4. Verify: `ping 10.66.0.1` с cli; `wg show`.
5. Прогоните один сервис через WG (например `curl http://10.66.0.1:8080/` или dig через WG IP).

Evidence: `20-wg-show.txt` (без приватных ключей в notes!).

### 4. Change — zones / egress

Минимум:

1. Интерфейс lab и `wg0` в разных зонах **или** отдельных nft chains с комментариями.
2. Входящий SSH/HTTPS только из lab (+ при желании из WG).
3. Egress с `srv`: разрешите обновления репо (http/https) **или** временно запретите произвольный egress и разрешите только DNS+NTP — зафиксируйте выбранную политику в `20-egress.md`.
4. Негатив: попытка запрещённого исходящего (например `curl` на заблокированный IP) — отказ; затем верните нужный доступ к репо.

| Семья | Ориентир |
|-------|----------|
| Rocky/Alma | `firewall-cmd --permanent --zone=… --change-interface=…`; policy objects при наличии |
| Debian/Ubuntu | nftables table inet filter + комментарии зон; ufw только если уже используете — опишите ограничения |

### 5. Verify — сводный сценарий

С `cli`:

1. `ping srv.lab` (DNS);
2. `ping` WG IP;
3. SSH по lab IP всё ещё работает (break-glass path).

Хронология в `20-verify.md`.

### 6. Automate

`lab-advnet-smoke.sh` на cli: dig srv.lab, ping WG peer, exit 1 при fail.

### 7. Document

Карта имён, WG addressing, зоны, egress-исключения, долги (нет split-DNS prod, нет MFA на WG).

## Критерии приёмки

- [ ] DNS lab резолвит srv/cli (и VIP при наличии)
- [ ] WireGuard up; ping между peers
- [ ] Зоны/policy описаны и применены
- [ ] Egress control с негативным тестом
- [ ] SSH break-path по lab Ethernet сохранён
- [ ] Smoke зелёный; ключи не в публичных notes

## Подсказки

- Endpoint WG = lab IP; не путайте с AllowedIPs.
- После смены DNS сохраните IP в inventory — при поломке резолва зайдёте по адресу.
- SELinux/AppArmor редко мешают WG; чаще — забытый UDP 51820 в firewall.

## Очистка

WG и dnsmasq можно оставить для капстоуна. При откате — верните resolv.conf/NM и `wg-quick down wg0`.
