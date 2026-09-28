# Лаборатория 18 — failover и карта SPOF

## Цель

Соберите один из двух стендов HA lite, явно уроните узел и докажите, что сервис остаётся доступен (или деградирует предсказуемо), затем задокументируйте оставшиеся SPOF.

## Окружение

- Минимум `srv` + `cli`
- Для ветки A (keepalived): удобна третья лёгкая ВМ **или** роль LB на `cli` + два backend (второй backend — контейнер/вторая ВМ). На 2 ВМ допустимо: VIP между `srv` и `cli`, если оба отдают один и тот же учебный HTTP.
- Снимок `before-ha`
- Lab L2: VIP должен быть в той же подсети, что интерфейсы ВМ

## Задания

### 1. Осмотр

```bash
ip -br a; ip route
curl -skI https://<srv>/ || curl -sI http://<srv>/
ss -tulpn | grep -E '80|443|nginx'
```

Нарисуйте текущий путь клиента → сервис и отметьте SPOF (`18-spof-before.md`).

### 2. Изменение — выберите ветку

#### Ветка A — keepalived + VIP

1. Одинаковый учебный HTTP(S) на двух нодах (или nginx на обеих с одним `index`).
2. `keepalived` на обеих: `virtual_ipaddress`, `priority`, `advert_int`, простой `vrrp_script` (проверка `pidof nginx` / `curl localhost`).
3. С `cli`: `curl http://<VIP>/` (или HTTPS) стабильно отвечает.
4. Сохраните вывод: `ip a` на master с VIP → `18-vip-master.txt`.

#### Ветка B — nginx upstream failover

1. Два backend: например nginx на `srv` и второй инстанс на `cli:8080` (или podman).
2. На LB-хосте (часто `cli`): `upstream` с двумя server; `proxy_pass`; `max_fails=1 fail_timeout=10s`.
3. Health: `curl` на LB VIP/имя → 200.
4. Сохраните: конфиг upstream + копия `18-upstream.conf`.

### 3. Проверка — отказ узла (обязательно)

1. Остановите **один** backend / master (`systemctl stop nginx` или `ip link` / stop keepalived на master).
2. Сразу и через 15–30 с: `curl` на VIP/LB — ожидаете успех (или краткий обрыв + успех).
3. Хронометраж в `18-failover-drill.md` (detect/failover/recover).
4. Негатив: остановите **оба** backend — сервис мёртв; зафиксируйте как оставшийся класс отказа.
5. Поднимите узлы обратно; VIP/upstream снова зелёный.

### 4. Документ — карта SPOF после

`18-spof-after.md`:

- что устранили;
- что осталось (LB, диск, DNS, LDAP, единственный Prometheus…);
- почему этого достаточно для mid+ lab.

### 5. Автоматизация

`lab-ha-smoke.sh`: `curl -fsS` к VIP/LB URL, exit 1 при fail. Запуск с `cli`.

## Критерии приёмки

- [ ] Выбрана и собрана ветка A или B
- [ ] Happy-path через VIP/LB работает
- [ ] Отказ одного узла: сервис доступен (с замерами)
- [ ] Отказ обоих / LB — осознанный негатив
- [ ] Карта SPOF до/после
- [ ] Smoke-скрипт зелёный

## Подсказки

| Семья | Заметки |
|-------|---------|
| Debian/Ubuntu | `keepalived`, `nginx` |
| Rocky/Alma | то же; SELinux: `httpd`/`nginx` network connect при proxy |

Если гипервизор режет VRRP multicast — используйте ветку B или unicast keepalived (документируйте).

## Очистка

VIP/upstream можно оставить для капстоуна. Не оставляйте два master с одним VIP без разбора split-brain (в lab — один master).
