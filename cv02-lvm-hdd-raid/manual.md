# Cviceni 2

## Ziskani informaci o discich a souborovych systemech

```bash
cat /proc/partitions
cat /proc/mdstat
df -h | df -m | df -i
du -h | du -hs | du -m
mount
mount -a
mount /dev/sda1 /mnt/
fdisk -l /dev/sda
cfdisk /dev/sda
sfdisk -s
sfdisk -d /dev/sda | sfdisk /dev/sdb
/etc/fstab
lsmod
```

## Zakladni operace s FS

```bash
mkfs.ext4 /dev/sda1
mkfs.xfs /dev/sda2
mount /dev/sda2 /mnt/

resize2fs /dev/sda1
xfs_grow /dev/sda2

mkswap /dev/sda3
swapon /dev/sda3
```

## Prace s obrazy

```bash
dd if=/dev/zero of=/tmp/image bs=1M \
count=1024

losetup -f /tmp/image
mkfs.ext4 /dev/loop0
mount  /dev/loop0 /mnt/
```

## RAID

```
raid0 - linear / striping
raid1 - mirroring
raid5 - striping + parita
raid6 - striping + 2x parita

raid01 - raid0 + raid1
raid10 - raid1 + raid0
raid50 - raid5 + raid0
raid60 - raid6 + raid0
```

### Sprava RAIDu

```bash
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 \
	/dev/sda1 /dev/sdb1

mdadm -Cv /dev/md0 -l1 -n2 /dev/sd[ab]1

mdadm -Cv /dev/md1 -l5  -n5 /dev/sda1 /dev/sdb1 /dev/sdc1 \
	/dev/sdd1 /dev/sde1

mdadm -Cv /dev/md1 -l5  -n4 -x1 /dev/sda1 /dev/sdb1 /dev/sdc1 \
	/dev/sdd1 /dev/sde1

mdadm -Cv /dev/md1 -l5  -n3 /dev/sda1 /dev/sdb1 missing

mdadm --fail /dev/md0 /dev/sda1
mdadm --remove /dev/md0 /dev/sda1
mdadm --add /dev/md0 /dev/sdc1

mdadm /dev/md0 --fail /dev/sda1 --remove /dev/sda1 --add /dev/sdc1

 mdadm --add/dev/md0 /dev/sdc1
 mdadm --grow /dev/md0 -n3

cat /proc/mdstat
mdadm --detail /dev/md0

watch --interval=10 cat /proc/mdstat

mdadm --stop /dev/md0
mdadm --remove /dev/md0

mdadm --zero-superblock /dev/sda

```

## LVM

```bash
pvdisplay | pvscan | pvremove 
pvcreate /dev/md0
pvmove -v /dev/sda2 /dev/sdb2

vgdisplay | vgscan | vgchange | vgremove | 
vgcreate vg-data/dev/md0
vgextend vg-data /dev/md1
vgreduce vg-data /dev/md0

lvdisplay | lvscan | lvchange | lvresize | lvrename
lvcreate -L 30G -n db-data vg-data
lvextend -L 60G -n db-data vg-data 
resize2fs /dev/vg-data/db-data
lvcreate -L 30G -m1 -n db-data vg-data
lvcreate --size 100M --snapshot -n db-data-snap /dev/vg-data/db-data
lvremove /dev/vg-data/db-data-snap

mount /dev/vg-data/db-data /mnt/data1
mount /dev/vg-data/db-data-snap /mnt/data2

vgcfgbackup
vgcfgrestore
```
