# Лаборатория 12 — капстоун mid+

## Цель

Соберите стенд mid+, который можно заново объяснить и сдать. Добавьте **одно усиление** из модулей 16–20 (HA / identity / containers), переживите инцидент с postmortem и сдайте документ передачи смены (`HANDOFF.md`) с пакетом доказательств.

## Окружение

- Стенд после модулей 01–11, **13–15** и желательно **16–20**
- `cli` для внешних проверок, DR и Prometheus
- Снимок `capstone-final` в конце

## Задания

### Обязательный объём

1. **Идентичность:** hostname, inventory.  
2. **SSH:** только ключ, без root/password login.  
3. **Роли:** оператор + opslab/данные.  
4. **LVM** данные + автомонтирование.  
5. **Firewall** минимален, каждое правило обосновано.  
6. **HTTPS:** nginx + TLS, проверка с `cli` (`curl -vk https://…`).  
7. **NFS** в lab-сеть **или** раздел «почему без» с компенсирующим контролем.  
8. **MAC:** SELinux enforcing или AppArmor enabled; нет «отключено навсегда».  
9. **Метрики + алерты:** exporter + Prometheus (или зафиксированный аналог модуля 14); алерты disk/service/HTTP; доказательства срабатывания.  
10. **Стенд на Ansible:** inventory srv+cli; ключевые роли (packages/users/sshd/firewall/nginx+tls/timer) применены playbooks; есть доказательства `check` и заметка о handlers.  
11. **DR:** копия на другой хост + timed restore drill с числами RPO/RTO.  
12. **Health/logs:** timer health-check не отменяет пункт 9.  
13. **Усиление из модулей 16–20 (обязательно — выберите ≥1):**  
    - **HA** — VIP/keepalived или nginx upstream failover + failover drill ([модуль 18](../18-ha-reliability/)); **или**  
    - **Identity** — SSSD/LDAP (или FreeIPA-lite): `getent` + SSH доменным user + break-glass ([модуль 17](../17-identity-sssd/)); **или**  
    - **Containers** — host run+volume+publish **и** k8s lite Deploy+Service+probes ([модуль 19](../19-containers-orchestration/)).  
14. **`HANDOFF.md`** (документ передачи смены) + **`12-postmortem.md`**.  
15. **Пакет доказательств** `~/lab-notes/12-evidence/`.

«HTTPS потом» / «Ansible потом» / «усиление потом» — **не** уровень сдачи.

### Желательно (сильно рекомендуется)

- Baseline/пороги из [16](../16-performance-capacity/) в `HANDOFF.md` или в заметках к алертам.  
- DNS lab / WireGuard / зоны из [20](../20-advanced-networking/), если уже собраны.  
- Второе усиление сверх обязательного выбора.

### Порядок работы

#### A. Осмотр — список пробелов

`12-gap-list.md` по обязательным пунктам (включая выбранное усиление 17/18/19). Снимок `12-before/`.

#### B. Изменение — закрытие пробелов

По одному; после Ansible-правок — `check` + apply. Не переписывайте стенд вручную в обход playbooks без записи drift.

#### C. Проверка — сдача с `cli`

Сохраните `12-acceptance.txt`:

```bash
ping -c 2 <IP-srv>
nc -vz <IP-srv> 22
nc -vz <IP-srv> 443
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@<IP-srv>  # отказ
ssh … 'hostname; findmnt /srv/data; curl -sk https://127.0.0.1/ | head'
curl -vk https://<IP-srv>/ 2>&1 | head -40
# NFS mount при наличии
# Prometheus: target srv UP; один alert rule виден
# Усиление: curl VIP/LB  ИЛИ  getent+ssh domain user  ИЛИ  kubectl/curl NodePort
```

#### D. Изменение — мини-инцидент + postmortem

Один сценарий (воспроизвести → устранить → postmortem):

| # | Инцидент |
|---|----------|
| 1 | Сломан fstab UUID для данных |
| 2 | Убран allow SSH в firewall |
| 3 | Сломан nginx TLS/конфиг |
| 4 | Алерт disk/service — довести до firing и погасить |
| 5 | Бэкап на другой хост просрочен > RPO |
| 6 | Отказ одного HA backend / LDAP down без break-glass check / pod NotReady |

`12-incident.md` (хронология) + `12-postmortem.md` (структура из theory).

#### E. Автоматизация — пакет доказательств

`12-evidence/`: inventory, sshd hardening, firewall, findmnt, tls-curl, nfs/exports или обоснование «почему без», mac-status, prometheus-targets, alert-drill, ansible-check-apply, backup-restore-drill (RPO/RTO), timers, **доказательства усиления** (ha/identity/k8s), incident, postmortem.

По желанию `lab-capstone-gather.sh`.

#### F. Документ — `HANDOFF.md`

Ответы на вопросы theory + ссылки на playbooks, alert rules и выбранный модуль 17/18/19.

#### G. Финал

Снимок `capstone-final`.

## Критерии приёмки (рубрика mid+)

| Критерий | Вес | Доказательство |
|----------|-----|----------------|
| SSH hardened | обязательно | отказ пароля + вход ключом |
| HTTPS с cli | обязательно | `curl -vk` / openssl |
| LVM данные | обязательно | findmnt, lsblk |
| Firewall минимален | обязательно | rules + обоснование |
| MAC не выключен | обязательно | getenforce / aa-status |
| Метрики + алерты | обязательно | target UP + alert drill |
| Стенд на Ansible | обязательно | playbooks + check/apply, сохранённый вывод |
| DR на другой хост + RPO/RTO | обязательно | заметки drill с числами |
| Усиление: HA **или** identity **или** containers | обязательно | drill/getent/kubectl — по выбору |
| Мини-инцидент | обязательно | incident + повторная проверка с `cli` |
| Postmortem | обязательно | `12-postmortem.md` |
| Пакет доказательств | обязательно | каталог `12-evidence/` |
| `HANDOFF.md` | обязательно | полные ответы |
| NFS или эквивалент | обязательно | mount или обоснование «почему без» |
| Пороги perf (16) | желательно | ссылка в `HANDOFF.md` / алертах |
| Adv net (20) | желательно | DNS/WG/zones если есть |
| Список пробелов | желательно | закрыт |

## Подсказки

- Оцениваются аккуратность и порядок работы, не «красота» HTML.
- Drift вручную после Ansible — долг в `HANDOFF.md`.
- Не светите учебные пароли и WG private keys в публичных копиях документа передачи смены.
- Ссылки на усиления: [16](../16-performance-capacity/), [17](../17-identity-sssd/), [18](../18-ha-reliability/), [19](../19-containers-orchestration/), [20](../20-advanced-networking/).

## Очистка

Стенд сохраните как портфолио.
