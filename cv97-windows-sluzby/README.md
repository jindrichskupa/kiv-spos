# Windows Server – Správa služeb

## 1. Úvod
**Cíl:** Seznámení s klíčovými službami operačního systému Windows Server a jejich konfigurací. Cvičení se zaměřuje na porozumění rolím jako Active Directory, DHCP, DNS, IIS a správě souborových služeb.

## 2. Použité nástroje a reference
*   **Windows Server:** [Homepage](https://www.microsoft.com/en-us/windows-server), [Dokumentace](https://docs.microsoft.com/en-us/windows-server/)
*   **Referenční materiál:** <http://home.zcu.cz/~pesicka/spos/WindowsCV2015.pdf> (Primární studijní materiál pro toto cvičení)

## 3. Poznámky ke cvičení (How-To)
_Vzhledem k tomu, že primární studijní materiál je externí PDF dokument, tato sekce poskytuje obecný přehled témat, která jsou typicky pokryta v rámci správy Windows Server služeb. Pro detailní postupy a příkazy se prosím řiďte referenčním PDF dokumentem._

### Přehled a správa rolí a funkcí
*   **Instalace rolí:** Přidání a odstranění serverových rolí (např. Active Directory Domain Services, DNS Server, DHCP Server, Web Server (IIS), File and Storage Services).
*   **Správa služeb:** Použití konzole `services.msc` pro spouštění, zastavování a konfiguraci spouštění jednotlivých služeb.

### Active Directory Domain Services (AD DS)
*   **Základy AD DS:** Domény, doménové řadiče, uživatelské účty, počítačové účty, organizační jednotky (OU).
*   **Instalace a konfigurace:** Povýšení serveru na doménový řadič.
*   **Správa uživatelů a počítačů:** Použití nástroje "Active Directory Users and Computers".

### DNS Server
*   **Instalace DNS role:** Integrace s Active Directory.
*   **Konfigurace zón:** Dopředné (Forward Lookup Zones) a reverzní (Reverse Lookup Zones) zóny.
*   **Záznamy DNS:** A, PTR, MX, CNAME záznamy.

### DHCP Server
*   **Instalace DHCP role:** Autorizace DHCP serveru v Active Directory.
*   **Konfigurace oborů (Scopes):** Rozsahy IP adres, vyloučení, rezervace, možnosti oboru.
*   **Správa pronájmů:** Sledování přidělených IP adres.

### Web Server (IIS)
*   **Instalace IIS role:** Základní instalace a ověření.
*   **Správa webových stránek:** Tvorba nových webů, nastavení bindingů, SSL certifikátů.
*   **Aplikační pooly:** Izolace webových aplikací.

### File Services
*   **Sdílené složky:** Konfigurace síťových sdílených složek (SMB/CIFS).
*   **Přístupová oprávnění:** Nastavení oprávnění pro sdílené složky a NTFS oprávnění.

### Remote Desktop Services (RDS)
*   **Přehled RDS:** Vzdálený přístup k aplikacím a plochám.
*   **Instalace role:** Základní komponenty RDS.

## 4. Příklad k procvičení

### Zadání

1.  Nainstalujte na Windows Server roli **DHCP Server**.
2.  Autorizujte DHCP server v doméně (pokud je server členem domény, jinak tento krok přeskočte).
3.  Vytvořte nový obor (Scope) s následujícími parametry:
    *   **Název oboru:** `SPOS_LAN`
    *   **Rozsah IP adres:** `192.168.10.100` až `192.168.10.199`
    *   **Maska podsítě:** `255.255.255.0`
    *   **Výchozí brána:** `192.168.10.1`
    *   **DNS servery:** `8.8.8.8`, `8.8.4.4`
    *   **Délka pronájmu:** 8 dní
4.  Aktivujte obor.
5.  Nainstalujte na Windows Server roli **DNS Server**.
6.  Vytvořte novou primární dopřednou zónu pro doménu `mujcviceni.local`.
7.  V této zóně vytvořte A záznam `server1` s IP adresou `192.168.10.10`.
8.  Nainstalujte na Windows Server roli **Web Server (IIS)**.
9.  Vytvořte novou webovou stránku v IIS, která bude obsluhovat obsah z adresáře `C:\inetpub\wwwroot\mojeweb` a bude dostupná na portu 8080. Do adresáře `mojeweb` vložte jednoduchý `index.html` soubor.

### Ověření
*   Zkuste získat IP adresu z DHCP serveru na klientském stroji. Ověřte, zda klient dostal IP adresu z definovaného rozsahu a správné DNS servery/bránu.
*   Na klientském stroji ověřte překlad `server1.mujcviceni.local` na `192.168.10.10` pomocí `nslookup` nebo `ping`.
*   Zkuste přistoupit k nově vytvořené webové stránce na portu 8080 z klientského stroje.

## 5. Doplňující materiály
*   **Prezentace/Cvičení Windows Server:** <http://home.zcu.cz/~pesicka/spos/WindowsCV2015.pdf>
