# Теория — DNS, WireGuard, зоны

## DNS-as-service (lab)

В модуле 07 клиент использовал чужой/провайдерский DNS. Здесь `srv` становится резолвером для lab:

| Стек | Роль |
|------|------|
| **dnsmasq** | DHCP+DNS lite, простые A-записи `srv.lab` / `cli.lab` |
| **Unbound** | рекурсор/форвардер, жёстче контроль |

Клиенты (`cli`, сам `srv`) указывают `nameserver <IP-srv>`. Проверка: `dig +short srv.lab @<IP-srv>`.

## WireGuard srv↔cli

Point-to-point VPN в lab-сети (или «через» эмуляцию недоверенного сегмента):

```text
srv wg0 10.66.0.1/24  <->  cli wg0 10.66.0.2/24
```

Ключи: `wg genkey` / `pubkey`. AllowedIPs минимальны. Трафик management/SSH можно оставить на lab Ethernet; через WG гоняйте учебный сервис или DNS.

## Firewall zones / egress

Идея: не плоский «всё разрешено в lab», а:

- зона `lab` / `internal` — srv↔cli;
- зона `vpn` — интерфейс `wg0`;
- **egress control**: с `srv` наружу только DNS/HTTP(S) к репо **или** явный deny + исключения (учебный минимум).

Debian: nftables chains / policy; Ubuntu ufw — ограниченно для зон.  
Rocky/Alma: **firewalld** zones (`trusted`, `internal`, `public`) + rich rules / policies.

Связь с **07**: базовые правила → здесь политика и именованные зоны.  
Связь с **17/18**: DNS имена для LDAP/VIP; WG как отдельный path.
