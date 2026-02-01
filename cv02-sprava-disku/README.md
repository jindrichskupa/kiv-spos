# Cvičení 2: Správa disků (RAID a LVM)

## 1. Úvod

Toto cvičení je zaměřeno na pokročilou správu diskových úložišť v Linuxu. Prozkoumáme dvě klíčové technologie, které umožňují flexibilní a spolehlivou konfiguraci disků nad rámec jednoduchých diskových oddílů:

*   **Software RAID:** Vytváření redundantních diskových polí pomocí nástroje `mdadm` pro ochranu dat a zvýšení výkonu.
*   **Logical Volume Manager (LVM):** Správa logických svazků, která umožňuje dynamicky měnit velikost diskových oddílů, vytvářet snapshoty a jednodušeji spravovat úložný prostor.

## 2. Použité nástroje a reference

*   **mdadm:** Nástroj pro správu softwarového RAIDu v Linuxu.
    *   Dokumentace: [docs.kernel.org](https://docs.kernel.org/admin-guide/md.html)
    *   GitHub: [md-raid-utilities/mdadm](https://github.com/md-raid-utilities/mdadm)
*   **LVM2:** Sada nástrojů pro správu Logical Volume Manageru.
    *   Homepage: [sourceware.org/lvm2/](https://sourceware.org/lvm2/)
    *   GitLab: [lvmteam/lvm2](https://gitlab.com/lvmteam/lvm2)
*   **Základní diskové nástroje:** Součást balíku `util-linux`.
    *   `fdisk`, `sfdisk`, `mkfs`, `mount`, `losetup`

## 3. Poznámky ke cvičení (How-To)

### Diagnostika disků a filesystémů

```bash
# Zobrazení diskových oddílů
fdisk -l
cat /proc/partitions

# Zobrazení stavu RAID polí
cat /proc/mdstat

# Zobrazení připojených filesystémů a jejich využití
df -hT

# Zobrazení využití místa v adresářích
du -sh /cesta/k/adresari

# Zobrazení připojených zařízení
mount
```

### Diskové oddíly (`fdisk`, `cfdisk`)

Pro rozdělení fyzických disků na oddíly (partitions) se v Linuxu tradičně používají nástroje jako `fdisk` a `cfdisk`. Tyto oddíly pak mohou sloužit jako základ pro filesystémy, RAID pole nebo LVM fyzické svazky.

*   **`fdisk`**: Interaktivní nástroj pro správu diskových oddílů. Podporuje tabulky oddílů MBR (Master Boot Record) a GPT (GUID Partition Table). Je textově orientovaný a vyžaduje zadávání příkazů.
*   **`cfdisk`**: Uživatelsky přívětivější varianta `fdisk` s pseudografickým rozhraním (curses based). Zjednodušuje vytváření a správu oddílů.

Před použitím těchto nástrojů je důležité identifikovat správný diskový cíl (např. `/dev/sda`, `/dev/sdb`).

```bash
# Zobrazení všech diskových zařízení a jejich oddílů
fdisk -l

# Spuštění interaktivního fdisk pro disk /dev/sdb
# (Příkazy jako 'n' pro nový oddíl, 'p' pro primární, 'w' pro zápis změn)
fdisk /dev/sdb

# Spuštění interaktivního cfdisk pro disk /dev/sdc
cfdisk /dev/sdc

# Příklad vytvoření nového primárního oddílu na /dev/sdb
# (Interaktivně v fdisku: n -> p -> 1 -> [Enter] -> [Enter] -> w)
```

_Poznámka: Po vytvoření nebo změně diskových oddílů je někdy nutné informovat jádro o těchto změnách. To lze provést restartem systému nebo pomocí `partprobe` nebo `kpartx -a /dev/sdb` (pro loop zařízení)._

### Správa souborového systému `ext4`

`ext4` (fourth extended filesystem) je široce používaný žurnálovací souborový systém pro Linux, který nabízí vylepšený výkon, spolehlivost a podporu pro velké soubory a disky oproti svým předchůdcům.

```bash
# Vytvoření nového souborového systému ext4 na oddílu /dev/sdb1
# Použijte s opatrností, smaže všechna data na oddílu!
mkfs.ext4 /dev/sdb1

# Kontrola souborového systému ext4 na chyby
e2fsck /dev/sdb1

# Změna velikosti ext4 souborového systému (po zvětšení underlying device)
# Nejprve zvětšete oddíl (např. pomocí fdisk/cfdisk nebo lvextend), pak FS
resize2fs /dev/sdb1

# Připojení souborového systému ext4 do adresáře /mnt/data
mkdir -p /mnt/data
mount /dev/sdb1 /mnt/data

# Odpojení souborového systému
umount /dev/sdb1

# Zobrazení informací o souborovém systému
dumpe2fs -h /dev/sdb1

# Změna popisku (labelu) souborového systému
e2label /dev/sdb1 my_ext4_label
```

### Správa souborového systému `ext4`

`ext4` (fourth extended filesystem) je široce používaný žurnálovací souborový systém pro Linux, který nabízí vylepšený výkon, spolehlivost a podporu pro velké soubory a disky oproti svým předchůdcům.

```bash
# Vytvoření nového souborového systému ext4 na oddílu /dev/sdb1
# Použijte s opatrností, smaže všechna data na oddílu!
mkfs.ext4 /dev/sdb1

# Kontrola souborového systému ext4 na chyby
e2fsck /dev/sdb1

# Změna velikosti ext4 souborového systému (po zvětšení underlying device)
# Nejprve zvětšete oddíl (např. pomocí fdisk/cfdisk nebo lvextend), pak FS
resize2fs /dev/sdb1

# Připojení souborového systému ext4 do adresáře /mnt/data
mkdir -p /mnt/data
mount /dev/sdb1 /mnt/data

# Odpojení souborového systému
umount /dev/sdb1

# Zobrazení informací o souborovém systému
dumpe2fs -h /dev/sdb1

# Změna popisku (labelu) souborového systému
e2label /dev/sdb1 my_ext4_label
```

### Software RAID (`mdadm`)

RAID (Redundant Array of Independent Disks) umožňuje spojit několik fyzických disků do jednoho logického celku.
*   **RAID 0 (Striping):** Zvyšuje výkon, ale nemá žádnou redundanci.
*   **RAID 1 (Mirroring):** Zrcadlí data na více discích pro vysokou redundanci.
*   **RAID 5 (Striping with Parity):** Vyžaduje min. 3 disky, poskytuje dobrou rovnováhu mezi výkonem a redundancí.
*   **RAID 6 (Striping with Double Parity):** Jako RAID 5, ale s dvojitou paritou, což umožňuje výpadek až dvou disků.

```bash
# Vytvoření RAID 1 pole ze dvou disků
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1

# Vytvoření RAID 5 pole se spare diskem
mdadm --create -v /dev/md1 -l5 -n4 -x1 /dev/sd[a-e]1

# Zobrazení detailů o RAID poli
mdadm --detail /dev/md0

# Simulace výpadku, odebrání a přidání disku
mdadm /dev/md0 --fail /dev/sda1
mdadm /dev/md0 --remove /dev/sda1
mdadm /dev/md0 --add /dev/sdc1

# Zastavení a smazání RAID pole
mdadm --stop /dev/md0
mdadm --remove /dev/md0
mdadm --zero-superblock /dev/sda1
```

### Logical Volume Manager (LVM)

LVM poskytuje flexibilní vrstvu nad fyzickými disky a RAID poli. Architektura:
*   **Physical Volume (PV):** Fyzický disk, diskový oddíl nebo RAID pole.
*   **Volume Group (VG):** Skupina jednoho nebo více PV, která tvoří jednotný úložný prostor.
*   **Logical Volume (LV):** "Virtuální" oddíl vytvořený z VG, který lze formátovat a používat.

```bash
# === Physical Volumes (PV) ===
# Inicializace disku jako PV
pvcreate /dev/sdb1

# Zobrazení PV
pvdisplay
pvscan

# === Volume Groups (VG) ===
# Vytvoření VG "vg-data" ze dvou PV
vgcreate vg-data /dev/sdb1 /dev/sdc1

# Rozšíření VG o další PV
vgextend vg-data /dev/sdd1

# Zobrazení VG
vgdisplay
vgscan

# === Logical Volumes (LV) ===
# Vytvoření LV "lv-web" o velikosti 10G z VG "vg-data"
lvcreate -L 10G -n lv-web vg-data

# Formátování a připojení LV
mkfs.ext4 /dev/vg-data/lv-web
mount /dev/vg-data/lv-web /var/www

# Zvětšení LV na 20G a následné zvětšení filesystému
lvextend -L 20G /dev/vg-data/lv-web
resize2fs /dev/vg-data/lv-web

# Vytvoření snapshotu (read-only "snímek" LV pro zálohy)
lvcreate --size 1G --snapshot -n web-snap /dev/vg-data/lv-web
```

### Práce s obrazy disků
Pro testování lze použít soubory jako obrazy disků.

```bash
# Vytvoření 100MB souboru
dd if=/dev/zero of=/tmp/image bs=1M count=100

# Připojení souboru jako loop device
losetup /dev/loop0 /tmp/image

# /dev/loop0 lze nyní použít jako běžný disk (např. v pvcreate)
```

## 4. Příklad k procvičení

### Zadání
1.  Vytvořit degradované pole `raid6` z disků `xvd{b,c,d}` (výsledná kapacita bude 200MB).
2.  Vytvořit soubor `/image` o velikosti 100MB, připojit jej jako loop device a označit jako LVM Physical Volume (PV).
3.  Vytvořit Volume Group `spos` z pole `/dev/md0` a loop zařízení.
4.  Vytvořit dva Logical Volume: `ext4` (100MB) a `xfs` (100MB).
5.  Naformátovat LV `ext4` na filesystém `ext4` a LV `xfs` na filesystém `xfs`.
6.  Připojit LV `ext4` do `/mnt/ext4`, LV `xfs` do `/mnt/xfs` a v každém vytvořit 1000 souborů.

### Ověření
Správnost nastavení si můžete ověřit pomocí následujících příkazů.

```bash
# 1. Ověření RAID pole
cat /proc/mdstat | grep md0 | grep raid6 && echo "RAID6 OK" || echo "RAID6 ERR"
fdisk -l /dev/md0  | grep MiB | awk '{if ($3 > 190) print "RAID SIZE OK"; else print "RAID SIZE ERR"}'

# 2. Ověření image souboru
test -f /image && echo "IMAGE OK" || echo "IMAGE ERR"

# 3. Ověření Volume Group a Physical Volumes
pvs | grep loop0 | grep spos && echo "LVM PV (loop0): OK" || echo "LVM PV (loop0): ERR"
pvs | grep md0 | grep spos && echo "LVM PV (md0): OK" || echo "LVM PV (md0): ERR"

# 4. Ověření Logical Volumes
lvs | grep ext4 | grep spos | grep '100.00m' && echo "LVM LV (ext4): OK" || echo "LVM LV (ext4): ERR"
lvs | grep xfs | grep spos | grep '100.00m' && echo "LVM LV (xfs): OK" || echo "LVM LV (xfs): ERR"

# 5. a 6. Ověření filesystémů a připojení
mount | grep spos-ext4 | grep /mnt/ext4 && echo "MOUNT ext4 OK" || echo "MOUNT ext4 ERR"
mount | grep spos-xfs | grep /mnt/xfs && echo "MOUNT xfs OK" || echo "MOUNT xfs ERR"
df -T | grep /mnt/ext4 | grep ext4 && echo "FS ext4 OK" || echo "FS ext4 ERR"
df -T | grep /mnt/xfs | grep xfs && echo "FS xfs OK" || echo "FS xfs ERR"

# 6. Ověření počtu souborů
[ $(find /mnt/ext4 -type f | wc -l) -eq 1000 ] && echo "FILES ext4 OK" || echo "FILES ext4 ERR"
[ $(find /mnt/xfs -type f | wc -l) -eq 1000 ] && echo "FILES xfs OK" || echo "FILES xfs ERR"
```

## 5. Doplňující materiály

### 5.1 Správa souborového systému `xfs`

`XFS` je vysoce výkonný žurnálovací souborový systém vyvinutý společností Silicon Graphics, který je vhodný pro systémy s velkými soubory a velkými objemy dat.

```bash
# Vytvoření nového souborového systému XFS na oddílu /dev/sdb2
mkfs.xfs /dev/sdb2

# Kontrola souborového systému XFS (obvykle se spouští automaticky při mountu)
xfs_repair /dev/sdb2

# Změna velikosti XFS souborového systému (po zvětšení underlying device)
# XFS lze zvětšovat pouze za chodu a pouze online
xfs_growfs /mnt/data_xfs

# Připojení souborového systému XFS do adresáře /mnt/data_xfs
mkdir -p /mnt/data_xfs
mount /dev/sdb2 /mnt/data_xfs

# Odpojení souborového systému
umount /dev/sdb2
```

### 5.2 Správa souborového systému `zfs`

`ZFS` je pokročilý souborový systém a správce logických svazků s funkcemi jako jsou datové integrity, snímky (snapshots), klonování a správa objemů. `ZFS` operuje na úrovni "zpool", což je fond úložiště složený z disků.

```bash
# Vytvoření zpool 'my_pool' z disku /dev/sdb
zpool create my_pool /dev/sdb

# Vytvoření souborového systému zfs 'my_pool/data'
zfs create my_pool/data

# Připojení souborového systému zfs (automaticky se připojí na /my_pool/data)
# zfs mount my_pool/data

# Vytvoření snímku (snapshotu)
zfs snapshot my_pool/data@yesterday

# Zobrazení zfs souborových systémů
zfs list

# Export zpoolu (pro odpojení disků a přenos)
zpool export my_pool

# Import zpoolu
zpool import my_pool
```

_Poznámka: `ZFS` není standardní součástí Linuxového jádra a vyžaduje instalaci dodatečných modulů a nástrojů. Na rozdíl od jiných FS zde není přímá analogie s `mkfs` nebo `mount` na jednotlivých oddílech, vše je řízeno přes `zpool` a `zfs` příkazy._

### 5.3 Správa souborového systému `btrfs`

`Btrfs` (B-tree file system) je moderní copy-on-write (CoW) souborový systém pro Linux, který nabízí pokročilé funkce jako jsou snímky (snapshots), subvolumy, integrita dat, komprese a RAID funkcionalita přímo v souborovém systému.

```bash
# Vytvoření nového souborového systému btrfs na oddílu /dev/sdb3
mkfs.btrfs /dev/sdb3

# Kontrola souborového systému btrfs (vyžaduje unmount)
btrfs scrub start /mnt/data_btrfs # Kontrola integrity dat
btrfs check /dev/sdb3 # Kontrola a oprava metadata (pouze offline)

# Změna velikosti btrfs souborového systému
# Lze zvětšovat i zmenšovat online
btrfs filesystem resize max /mnt/data_btrfs

# Připojení souborového systému btrfs do adresáře /mnt/data_btrfs
mkdir -p /mnt/data_btrfs
mount /dev/sdb3 /mnt/data_btrfs

# Vytvoření subvolumu
btrfs subvolume create /mnt/data_btrfs/mysubvolume

# Vytvoření snímku (snapshotu) subvolumu
btrfs subvolume snapshot /mnt/data_btrfs/mysubvolume /mnt/data_btrfs/mysubvolume_snap

# Odpojení souborového systému
umount /dev/sdb3
```