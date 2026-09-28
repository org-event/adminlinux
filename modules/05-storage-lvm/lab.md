# Лаборатория 05 — диски и LVM

## Цель

Подключить второй диск, собрать LVM, настроить автомонтирование `/srv/data` и отработать типичные инциденты (сломанный fstab, заполнение тома, расширение).

## Окружение

- ВМ `srv` + **второй виртуальный диск** 5–10 ГБ
- Снимок `before-lvm` **до** любых правок дисков
- Консоль гипервизора открыта (на случай проблем с загрузкой после fstab)

## Задания

### 1. Inspect — карта блочных устройств

```bash
lsblk -f
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,UUID
sudo fdisk -l
df -hT
findmnt
```

1. Найдите **новый** диск (часто `/dev/sdb` или `/dev/vdb`). **Не трогайте** системный диск с `/`.
2. Сохраните «до» в `~/lab-notes/05-before.txt` (`lsblk -f`, `df -hT`).
3. В заметках явно напишите: «системный диск = …, дополнительный = …».

### 2. Change — раздел и LVM

1. Создайте один раздел на новом диске (тип Linux LVM, если спрашивает `fdisk`/`parted`).
2. Соберите PV → VG `vgdata` → LV `lvapp` размером **~2G** (оставьте свободное место в VG ≥ 2G).
3. Создайте filesystem: `ext4` (Debian/Ubuntu) или `xfs` (Rocky/Alma — допустимо).
4. Смонтируйте в `/srv/data`, создайте тестовый файл `hello-lvm.txt`.

```bash
# пример каркаса (адаптируйте имя устройства!)
sudo pvcreate /dev/sdb1
sudo vgcreate vgdata /dev/sdb1
sudo lvcreate -n lvapp -L 2G vgdata
sudo mkfs.ext4 /dev/vgdata/lvapp   # или mkfs.xfs
sudo mkdir -p /srv/data
sudo mount /dev/vgdata/lvapp /srv/data
echo "ok $(date -Is)" | sudo tee /srv/data/hello-lvm.txt
sudo vgs; sudo lvs; sudo pvs
```

### 3. Change — fstab по UUID

1. Получите UUID: `sudo blkid /dev/vgdata/lvapp`.
2. Добавьте запись в `/etc/fstab` **только по UUID** (не по `/dev/sdX`).
3. Проверка без ребута:

```bash
sudo umount /srv/data
sudo mount -a
findmnt /srv/data
df -h /srv/data
```

4. **Перезагрузите** ВМ и убедитесь, что `/srv/data` снова на месте.

### 4. Change — расширение LV

Расширьте LV ещё на **+1G** и растяните ФС:

```bash
# ext4
sudo lvextend -L +1G /dev/vgdata/lvapp
sudo resize2fs /dev/vgdata/lvapp

# xfs (том должен быть смонтирован)
sudo lvextend -L +1G /dev/vgdata/lvapp
sudo xfs_growfs /srv/data
```

Зафиксируйте `df -h /srv/data` до/после в evidence.

### 5. Verify — негативный тест fstab (мини-инцидент)

**Только на снимке / с консолью гипервизора.**

1. Сделайте копию: `sudo cp /etc/fstab /etc/fstab.bak.lab05`.
2. Временно испортите UUID в строке `/srv/data` (одна цифра).
3. Выполните `sudo mount -a` — зафиксируйте ошибку в `~/lab-notes/05-fstab-incident.txt`.
4. Восстановите из `.bak` и снова `mount -a`.
5. В заметках опишите: чем опасен `nofail` vs без него при ошибке UUID.

### 6. Verify — заполнение тома (мини-инцидент)

1. Создайте файл, почти заполняющий `/srv/data` (`fallocate` или `dd`), пока `df` не покажет высокое использование.
2. Попробуйте создать ещё один маленький файл — зафиксируйте симптом.
3. Удалите тестовый «мусор», освободите место.
4. В `~/lab-notes/05.md` ответьте: чем отличается «диск полный» от «inode закончились» (`df -i`).

### 7. Automate — проверка монтирования одной командой

Создайте `/usr/local/bin/lab-check-data-mount.sh`:

- проверяет, что `/srv/data` смонтирован с LVM (`findmnt`);
- проверяет наличие `hello-lvm.txt` (или создаёт marker);
- пишет OK/FAIL в `/var/log/lab-mount-check.log`;
- код возврата 0/1.

Запустите вручную и сохраните вывод.

### 8. Document

В `~/lab-notes/05.md` — схема: диск → раздел → PV → VG → LV → ФС → mountpoint → UUID.  
В `~/lab-notes/05-evidence.txt` — `lsblk -f`, `vgs`/`lvs`, `df -h /srv/data`, `findmnt /srv/data`, фрагмент `/etc/fstab`.

## Критерии приёмки

- [ ] `/srv/data` смонтирован с LVM-тома
- [ ] После `umount` + `mount -a` и после reboot том возвращается
- [ ] В VG было свободное место и LV расширен (+ доказательство df)
- [ ] Негативный тест fstab проведён и откатан
- [ ] Мини-инцидент «диск полный» отработан
- [ ] Есть скрипт проверки монтирования
- [ ] Есть схема и evidence
- [ ] Системный диск не затронут

## Подсказки

| Тема | Debian/Ubuntu | Rocky/Alma |
|------|---------------|------------|
| ФС по умолчанию в примерах | `ext4` + `resize2fs` | часто `xfs` + `xfs_growfs` |
| Пакеты LVM | `lvm2` | `lvm2` |

- Имена `/dev/sdX` меняются — в fstab только UUID.
- Если ВМ не видит диск — проверьте гипервизор, затем перескан/reboot.
- `mount -a` на проде опасен без бэкапа fstab — у вас учебный стенд и снимок.

## Очистка

Не удаляйте том — он нужен для бэкапов в модуле 11. Откат — только через снимок `before-lvm`.
