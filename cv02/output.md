# Cviceni 2 - vystup

## Zadani

1. Vytvorit degradovany raid6 z disku xvd{b,c,d} = vysledna kapacita bude 200MB
2. Vytvorit image disku /image o velikosti 100MB, vytvorit loop device a oznavit ho jako PV
3. Vytvorit VG spos z md0 a loop0
4. Vytvorit dva oddily: ext4 100MB, xfs 100MB
5. Naformatovat oddil ext4 na fs ext4 a xfs na xfs
6. Pripojit oddil ext4 do /mnt/ext4, oddil xfs do /mnt/xfs a zalozit v kazdem 1000 souboru

## Kontrola

```bash
cat /proc/mdstat | grep md0 | grep raid6 && echo "RAID6 OK" || echo "RAID6 ERR"

fdisk -l /dev/md0  | grep MiB | awk '{if ($3 > 190) print "RAID SIZE OK"; else print "RAID SIZE ERR"}'

test -f /image && echo "IMAGE OK" || echo "IMAGE ERR"

#du -ms /image | awk '{if ($1 == 100) print "IMAGE SIZE OK"; else print "IMAGER SIZE ERR"}'

pvs | grep loop0 | awk '{if ($2=="spos" && $1 == "/dev/loop0") print "LVM /dev/loop0 - OK"; else print "LVM /dev/loop0 - ERR"}'
pvs | grep md0 | awk '{if ($2=="spos" && $1 == "/dev/md0") print "LVM /dev/md0 - OK"; else print "LVM /dev/md0 - ERR"}'

lvs | grep ext4 | awk '{if ($2=="spos" && $1 == "ext4" && $4 == "100,00m") print "LVM ext4 - OK"; else print "LVM ext4 - ERR"}'
lvs | grep xfs | awk '{if ($2=="spos" && $1 == "xfs" && $4 == "100,00m") print "LVM xfs - OK"; else print "LVM xfs - ERR"}'

mount | grep spos-ext4 | awk '{if ($3 == "/mnt/ext4") print "MOUNT ext4 OK"; else print "MOUNT ext4 ERR"}'
mount | grep spos-xfs | awk '{if ($3 == "/mnt/xfs") print "MOUNT xfs OK"; else print "MOUNT xfs ERR"}'

df -hT |grep /mnt/ext4 | tr -d M | awk '{if ($3 > 90 && $2 == "ext4") print "FS AND SIZE ext4 OK"; else print "FS AND SIZE ext4 ERR"}'
df -hT |grep /mnt/xfs | tr -d M | awk '{if ($3 > 90 && $2 == "xfs") print "FS AND SIZE xfs OK"; else print "FS AND SIZE xfs ERR"}'

[ $(find /mnt/ext4 -type f | wc -l) -eq 1000 ] && echo "FILES OK" || echo "FILES ERR"
[ $(find /mnt/xfs -type f | wc -l) -eq 1000 ] && echo "FILES OK" || echo "FILES ERR"
```

