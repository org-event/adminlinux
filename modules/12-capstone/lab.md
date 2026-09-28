# Лаборатория 12 — капстоун mid+

## Цель

Привести стенд к воспроизводимому mid+ состоянию, добавить **одно усиление** из эшелона 16–20 (HA / identity / containers), пережить инцидент с postmortem и сдать handoff с evidence pack.

## Окружение

- Стенд после модулей 01–11, **13–15** и желательно **16–20**
- `cli` для acceptance, DR, Prometheus
- Снимок `capstone-final` в конце

## Задания

### Обязательный объём (must)

1. **Идентичность:** hostname, inventory.  
2. **SSH:** ключ only, без root/password login.  
3. **Роли:** оператор + opslab/данные.  
4. **LVM** данные + автомонтирование.  
5. **Firewall** минимален, каждое правило обосновано.  
6. **HTTPS:** nginx + TLS, проверка с `cli` (`curl -vk https://…`).  
7. **NFS** в lab-сеть **или** раздел «почему без» с компенсирующим контролем.  
8. **MAC:** SELinux enforcing или AppArmor enabled; нет «отключено навсегда».  
9. **Metrics + alerts:** exporter + Prometheus (или зафиксированный аналог модуля 14); алерты disk/service/HTTP; evidence срабатывания.  
10. **Ansible-built:** inventory srv+cli; ключевые роли (packages/users/sshd/firewall/nginx+tls/timer) применены playbooks; есть `check` evidence и заметка о handlers.  
11. **DR:** off-host копия + timed restore drill с числами RPO/RTO.  
12. **Health/logs:** timer health-check не отменяет пункт 9.  
13. **Эшелон-усиление (must — выберите ≥1):**  
    - **HA** — VIP/keepalived или nginx upstream failover + failover drill ([модуль 18](../18-ha-reliability/)); **или**  
    - **Identity** — SSSD/LDAP (или FreeIPA-lite): `getent` + SSH доменным user + break-glass ([модуль 17](../17-identity-sssd/)); **или**  
    - **Containers** — host run+volume+publish **и** k8s lite Deploy+Service+probes ([модуль 19](../19-containers-orchestration/)).  
14. **HANDOFF.md** + **12-postmortem.md**.  
15. **Evidence pack** `~/lab-notes/12-evidence/`.

«HTTPS optional» / «Ansible optional» / «эшелон потом» — **не** уровень сдачи.

### Should (сильно рекомендуется)

- Baseline/пороги из [16](../16-performance-capacity/) в handoff или alert notes.  
- DNS lab / WireGuard / zones из [20](../20-advanced-networking/), если уже собраны.  
- Второе эшелон-усиление сверх must-выбора.

### Порядок работы

#### A. Inspect — gap-list

`12-gap-list.md` по must-пунктам (включая выбранное усиление 17/18/19). Снимок `12-before/`.

#### B. Change — закрытие gaps

По одному; после Ansible-правок — `check` + apply. Не переписывайте стенд вручную в обход playbooks без записи drift.

#### C. Verify — acceptance с `cli`

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
# Эшелон: curl VIP/LB  ИЛИ  getent+ssh domain user  ИЛИ  kubectl/curl NodePort
```

#### D. Change — мини-инцидент + postmortem

Один сценарий (воспроизвести → устранить → postmortem):

| # | Инцидент |
|---|----------|
| 1 | Сломан fstab UUID для данных |
| 2 | Убран allow SSH в firewall |
| 3 | Сломан nginx TLS/конфиг |
| 4 | Алерт disk/service — довести до firing и погасить |
| 5 | Off-host бэкап просрочен > RPO |
| 6 | Отказ одного HA backend / LDAP down без break-glass check / pod NotReady |

`12-incident.md` (хронология) + `12-postmortem.md` (структура из theory).

#### E. Automate — evidence pack

`12-evidence/`: inventory, sshd hardening, firewall, findmnt, tls-curl, nfs/exports или rationale, mac-status, prometheus-targets, alert-drill, ansible-check-apply, backup-restore-drill (RPO/RTO), timers, **эшелон-evidence** (ha/identity/k8s), incident, postmortem.

Опционально `lab-capstone-gather.sh`.

#### F. Document — HANDOFF

Ответы на вопросы theory + ссылки на playbooks, alert rules и выбранный модуль 17/18/19.

#### G. Финал

Снимок `capstone-final`.

## Критерии приёмки (рубрика mid+)

| Критерий | Вес | Доказательство |
|----------|-----|----------------|
| SSH hardened | must | отказ пароля + вход ключом |
| HTTPS с cli | must | `curl -vk` / openssl |
| LVM данные | must | findmnt, lsblk |
| Firewall минимален | must | rules + обоснование |
| MAC не выключен | must | getenforce / aa-status |
| Metrics + alerts | must | target UP + alert drill |
| Ansible-built | must | playbooks + check/apply evidence |
| Off-host DR + RPO/RTO | must | drill notes с числами |
| Эшелон: HA **или** identity **или** containers | must | drill/getent/kubectl evidence по выбору |
| Мини-инцидент | must | incident + повторный acceptance |
| Postmortem | must | `12-postmortem.md` |
| Evidence pack | must | каталог `12-evidence/` |
| HANDOFF | must | полные ответы |
| NFS или эквивалент | must | mount или rationale |
| Perf thresholds (16) | should | ссылка в handoff/alerts |
| Adv net (20) | should | DNS/WG/zones если есть |
| Gap-list | should | закрыт |

## Подсказки

- Оценивается дисциплина оператора, не «красота» HTML.
- Drift вручную после Ansible — долг в handoff.
- Не светите учебные пароли и WG private keys в публичных копиях handoff.
- Ссылки на модули эшелона: [16](../16-performance-capacity/), [17](../17-identity-sssd/), [18](../18-ha-reliability/), [19](../19-containers-orchestration/), [20](../20-advanced-networking/).

## Очистка

Стенд сохраните как портфолио.
