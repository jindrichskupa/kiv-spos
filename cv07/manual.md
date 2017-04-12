# Cviceni 7

## NFS

Instalace

```bash
apt -get install nfs-kernel-server
apt-get install nfs-server
```

Nastaveni

```bash
/etc/exports:
/srv/share1       10.228.67.0/24(rw) 147.228.67.0/24(ro)
/srv/share2       10.228.67.0/24(rw,no_root_squash)
/srv/share3       10.228.67.0/24(rw,no_subtree_check,
                            all_squash,anonuid=6000,anongid=6000)
```

Info

```bash
exportfs -a
exportfs -r
exportfs -u
```

Pripojeni

```bash
mount -t nfs  10.228.67.42:/srv/share1 /mnt/share1
mount -t nfs  -o ro 10.228.67.42:/srv/share1 /mnt/share1

/etc/fstab
```

Mapovani

```bash
addgroup --gid 6000 nfs
addgroup --gid 6000 --uid 6000 nfs
```

## Samba
 
Instalace 

```bash
apt-get install samba smbclient cifs-utils
```

Uzivatele

```bash
smbpasswd -a jindra
pdbedit -w -L
```

Skupina

```bash
addgroup --gid 6001 cifs
usermod -G cifs jindra
```

Klient

```bash
smbclient //localhost/share1 -U jindra
smbclient -L localhost
```

Nastaveni

```bash
/etc/samba/smb.conf:
server string = Samba Server %v
netbios name = debiansecurity = user
[share1]
        comment = Prvni share co jsem kdy vyrobili
        path = /srv/share1
        browsable = yes        writable = yes
        guest ok = yes
        create mask = 0600
        directory mask = 0700

[share2]
        path = /srv/share2
        valid users = @users
        read only = yes

[share3]
        path = /srv/share3
        valid users = jindra
[anonymous]
        path = /srv/anonymous
        force group = cifs
        create mask = 0660        directory mask = 0771        browsable =yes
        writable = yes
        guest ok = yes

```

Pripojeni

```bash
mount -t cifs //localhost/share1 /mnt/share -o username=jindra

/etc/fstab
```
