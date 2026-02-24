# Správa síťových disků (NFS a Samba)

## 1. Úvod
**Cíl:** Seznámení s konfigurací a správou síťových souborových systémů NFS (Network File System) pro Linux/Unix klienty a Samba (SMB/CIFS) pro Windows klienty. Cvičení zahrnuje nastavení sdílených adresářů, uživatelských práv, a připojení síťových disků na klientských systémech.

## 2. Použité nástroje a reference
*   **NFS (Network File System):** Protokol pro distribuovaný souborový systém.
    *   Wikipedia: [https://en.wikipedia.org/wiki/Network_File_System](https://en.wikipedia.org/wiki/Network_File_System)
    *   Oficiální dokumentace (Linux man pages): [man exports](https://linux.die.net/man/5/exports)
    *   **NFS Server Utilities:**
        *   **nfs-kernel-server:** Balíček obsahující NFS server.
        *   **exportfs:** Nástroj pro správu NFS exportů.
            *   Reference: [man exportfs](https://linux.die.net/man/8/exportfs)
    *   **NFS Client Utilities:**
        *   **mount:** Pro připojení NFS sdílených adresářů.
            *   Reference: [man mount](https://manpages.debian.org/mount)
*   **Samba:** Implementace protokolu SMB/CIFS pro sdílení souborů a tiskáren s Windows klienty.
    *   Homepage: [https://www.samba.org/](https://www.samba.org/)
    *   Dokumentace: [Samba Documentation](https://www.samba.org/samba/docs/)
    *   **Samba Server Utilities:**
        *   **samba:** Balíček obsahující Samba server (`smbd`, `nmbd`).
        *   **smbpasswd:** Správa hesel Samba uživatelů.
            *   Reference: [man smbpasswd](https://www.samba.org/samba/docs/current/manpages/smbpasswd.8.html)
        *   **pdbedit:** Správa databáze uživatelů Samby.
            *   Reference: [man pdbedit](https://www.samba.org/samba/docs/current/manpages/pdbedit.8.html)
    *   **Samba Client Utilities:**
        *   **smbclient:** Klient pro přístup k SMB/CIFS sdíleným adresářům.
            *   Reference: [man smbclient](https://www.samba.org/samba/docs/current/manpages/smbclient.1.html)
        *   **cifs-utils:** Nástroje pro připojení CIFS/SMB share (`mount.cifs`).
*   **Systemd:** Pro správu NFS a Samba služeb.
    *   Reference: [man systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html)
*   **Základní nástroje pro správu uživatelů a skupin:** `addgroup`, `adduser`, `usermod`.
    *   Reference: [man addgroup](https://manpages.debian.org/addgroup), [man adduser](https://manpages.debian.org/adduser), [man usermod](https://manpages.debian.org/usermod)

## 3. Poznámky ke cvičení (How-To)

### NFS

#### Instalace
```bash
apt-get install nfs-kernel-server
# Starší systémy nebo alternativní balíček:
# apt-get install nfs-server
```

#### Konfigurace (`/etc/exports`)
Soubor `/etc/exports` definuje adresáře, které budou sdíleny, a jaká přístupová práva k nim mají klienti.
```
/srv/share1       10.228.67.0/24(rw) 147.228.67.0/24(ro)
/srv/share2       10.228.67.0/24(rw,no_root_squash)
/srv/share3       10.228.67.0/24(rw,no_subtree_check,all_squash,anonuid=6000,anongid=6000)
```
*   `rw`: read/write přístup
*   `ro`: read-only přístup
*   `no_root_squash`: root na klientovi má práva roota na serveru (NEBEZPEČNÉ!)
*   `all_squash`: mapuje všechny uživatele na anonymního uživatele
*   `anonuid`, `anongid`: UID a GID pro anonymního uživatele

#### Správa NFS exportů
```bash
exportfs -a  # Exportuje všechny adresáře z /etc/exports
exportfs -r  # Znovu exportuje všechny adresáře (reload)
exportfs -u  # Zruší export (unexport)
```

#### Připojení NFS share na klientovi
```bash
# Ruční připojení
mount -t nfs 10.228.67.42:/srv/share1 /mnt/share1
# Read-only připojení
mount -t nfs -o ro 10.228.67.42:/srv/share1 /mnt/share1
```
Pro automatické připojení při startu systému přidejte záznam do `/etc/fstab`.

#### Mapování uživatelů/skupin (pro `all_squash`)
```bash
addgroup --gid 6000 nfsgroup
adduser --uid 6000 --gid 6000 --no-create-home --disabled-password nfsuser
```

### Samba

#### Instalace
```bash
apt-get install samba smbclient cifs-utils
```

#### Správa Samba uživatelů
```bash
smbpasswd -a jindra # Přidá Samba uživatele a nastaví heslo
pdbedit -w -L       # Vypíše seznam Samba uživatelů
```

#### Správa skupin (pro přístup k share)
```bash
addgroup --gid 6001 cifsgroup
usermod -G cifsgroup jindra # Přidá uživatele jindra do skupiny cifsgroup
```

#### Klient Samba
```bash
smbclient //localhost/share1 -U jindra # Připojení k share 'share1' jako uživatel 'jindra'
smbclient -L localhost                 # Vypsání dostupných share na localhostu
```

#### Konfigurace (`/etc/samba/smb.conf`)
Hlavní konfigurační soubor Samby. Sekce `[global]` obsahuje globální nastavení, sekce `[share_name]` definují jednotlivé sdílené adresáře.
```ini
# /etc/samba/smb.conf
[global]
server string = Samba Server %v
netbios name = debian
security = user

[share1]
        comment = Prvni share co jsem kdy vyrobili
        path = /srv/share1
        browsable = yes
        writable = yes
        guest ok = yes
        create mask = 0600
        directory mask = 0700

[share2]
        path = /srv/share2
        valid users = @users # Pouze členové systémové skupiny 'users'
        read only = yes

[share3]
        path = /srv/share3
        valid users = jindra # Pouze uživatel 'jindra'
        writable = yes

[anonymous]
        path = /srv/anonymous
        force group = cifs # Nové soubory/adresáře budou patřit skupině 'cifs'
        create mask = 0660
        directory mask = 0771
        browsable = yes
        writable = yes
        guest ok = yes
```
Po jakékoli změně v `smb.conf` je potřeba restartovat službu Samba: `systemctl restart smbd nmbd`.

#### Připojení Samba share na klientovi
```bash
# Ruční připojení
mount -t cifs //localhost/share1 /mnt/share -o username=jindra,password=heslo
```
Pro automatické připojení při startu systému přidejte záznam do `/etc/fstab` (doporučuje se použít soubor s heslem pro bezpečnost).

## 4. Příklad k procvičení

### Zadání

1.  Nasdílejte adresář `/srv/shared1` přes NFS i Sambu:
    *   **NFS:**
        *   Read-only pro síť `147.228.68.0/24`.
        *   Read-write pro síť `192.168.255.0/24`.
        *   Mapování na uživatele `nfsshare`.
    *   **Samba:**
        *   S možností zápisu.
        *   Dostupné pouze přes login `smbshare`.
2.  Připojte obě sdílení:
    *   **NFS:** do `/mnt/shared1nfs-ro` a `/mnt/shared1nfs-rw`.
    *   **Samba:** do `/mnt/shared1samba`.
3.  Ověřte nastavená práva.
4.  Připravte do systému automatické připojování po startu systému (zakomentované řádky v `/etc/fstab`).

### Ověření
*   Manuální kontrola práv zápisu/čtení z různých sítí/uživatelů.
*   Testování připojení NFS a Samba share na klientovi.
*   Kontrola obsahu `/etc/fstab`.

## 5. Doplňující materiály
_Žádné dodatečné doplňující materiály nejsou k dispozici v původních souborech._
