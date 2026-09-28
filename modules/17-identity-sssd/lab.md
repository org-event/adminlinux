# Лаборатория 17 — OpenLDAP + SSSD (lite)

## Цель

Поднять каталог на `srv`, подключить SSSD на `cli` (и по желанию на `srv`), доказать `getent`/SSH/группы/sudo и break-glass при отказе LDAP.

## Окружение

- `srv` + `cli`, lab-сеть
- RAM `srv` ≥1.5 ГБ (для FreeIPA-ветки — лучше ≥3 ГБ)
- Снимок `before-identity`
- Firewall: LDAP (`389`/`636`) **только** из lab / с IP `cli`

Рекомендуемый путь: **OpenLDAP + SSSD**. FreeIPA single-node — альтернатива с теми же критериями приёмки.

## Задания

### 1. Inspect

```bash
getent passwd | wc -l
grep -E 'passwd|group|shadow' /etc/nsswitch.conf
ss -tulpn | grep -E '22|389|636' || true
```

Зафиксируйте локальных admin-пользователей (break-glass кандидат).

### 2. Change — каталог на `srv`

**Ветка OpenLDAP (рекомендуется):**

1. Установите `slapd` / `openldap-servers` по семье дистрибутива.
2. База: суффикс вида `dc=lab,dc=local`, admin DN, учебный пароль.
3. Добавьте OU `People` / `Groups`, пользователя `alice`, группу `opslab` (GID согласован с политикой 03).
4. Проверка с srv: `ldapsearch -x -b dc=lab,dc=local '(uid=alice)'`.

**Ветка FreeIPA-lite:** `ipa-server-install` в lab (фиксируйте домен/hostname в DNS). Дальше критерии те же.

Evidence: `17-ldap-search.txt`.

### 3. Change — SSSD на `cli`

1. Пакеты `sssd` `sssd-ldap` (или `sssd-ipa`).
2. `/etc/sssd/sssd.conf` (mode `0600`): domain, `ldap_uri`, search base, TLS по возможности (`ldap_tls_reqcert` для lab — документируйте insecure, если self-signed).
3. `nsswitch`: `passwd/group` → `files sss`.
4. PAM: через `authselect` (Rocky/Alma) или `pam-auth-update` / пакетные профили (Debian/Ubuntu) — **не** правьте все файлы вручную без бэкапа.
5. `systemctl enable --now sssd`.

### 4. Verify — identity path

```bash
getent passwd alice
id alice
# SSH с cli или на cli:
ssh alice@cli   # или alice@srv — как спроектировали
```

Негатив: неизвестный `bob_not_exist` → отказ.

Evidence: `17-getent.txt`, `17-ssh-alice.txt`.

### 5. Change — sudo / groups

1. Доменную группу в `sudoers.d` **или** `ldap_sudo` / IPA HBAC+sudo (если IPA).
2. Проверка: `sudo -l -U alice` и одна безопасная команда.
3. Негатив: пользователь вне группы — sudo отказан (если так задумано).

### 6. Verify — break-glass + LDAP outage

1. Убедитесь: локальный `labadmin` (или аналог) входит по ключу и имеет sudo **без** LDAP.
2. Остановите slapd/ipa на `srv` (или firewall drop 389 с cli).
3. Документируйте: кэшированный `alice` ещё логинится или нет; `labadmin` точно логинится.
4. Поднимите каталог обратно.

`17-break-glass.md` — хронология.

### 7. Automate

`/usr/local/bin/lab-identity-smoke.sh` на `cli`:

- `getent passwd alice` успешен;
- `systemctl is-active sssd`;
- exit 1 иначе.

### 8. Document

Топология DN/URI, кто клиент, TLS-статус, break-glass процедура, долги (нет реплики, нет MFA).

## Критерии приёмки

- [ ] Каталог отвечает; `ldapsearch`/`ipa` evidence есть
- [ ] SSSD на клиенте; `getent` видит доменного user
- [ ] SSH доменным user работает
- [ ] sudo/groups согласованы и проверены
- [ ] Break-glass local admin доказан при outage
- [ ] Smoke-скрипт зелёный
- [ ] LDAP не торчит в «мир»

## Подсказки

| Семья | Заметки |
|-------|---------|
| Debian/Ubuntu | `slapd`, `ldap-utils`, `sssd`, `sssd-ldap`; часто `dpkg-reconfigure slapd` |
| Rocky/Alma | `openldap-servers`, `openldap-clients`, `sssd`, `authselect select sssd with-mkhomedir` |

SELinux: при AVC на slapd/sssd — модуль 13, не `setenforce 0` навсегда.  
Home: `oddjob-mkhomedir` / `pam_mkhomedir` — удобно для первого входа.

## Очистка

Каталог можно оставить для капстоуна. Снимок `after-identity`. Не удаляйте break-glass admin.
