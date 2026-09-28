# Теория — диски и LVM

## Блочные устройства

```bash
lsblk -f
sudo fdisk -l
```

Типичный путь: диск → раздел → (опционально LVM) → файловая система → точка монтирования.

## LVM словами админа

- **PV** (physical volume) — диск/раздел под LVM
- **VG** (volume group) — пул из PV
- **LV** (logical volume) — том, который форматируете и монтируете

Зачем: проще расширять тома, чем классические разделы.

```bash
sudo pvcreate /dev/sdb1
sudo vgcreate vgdata /dev/sdb1
sudo lvcreate -n lvapp -L 2G vgdata
sudo mkfs.ext4 /dev/vgdata/lvapp
sudo mkdir -p /srv/data
sudo mount /dev/vgdata/lvapp /srv/data
```

## fstab

Чтобы том монтировался после перезагрузки, добавьте запись в `/etc/fstab` **по UUID**:

```bash
sudo blkid
# UUID=...  /srv/data  ext4  defaults  0  2
```

Проверка без ребута:

```bash
sudo findmnt --verify
sudo mount -a
```

Ошибка в fstab может оставить систему в emergency mode — отсюда обязательный снимок.

## Место и inode

```bash
df -h
df -i
du -sh /var/*
```
