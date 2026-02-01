# Windows Server

## 1. Úvod
**Cíl:** Seznámení se základy správy operačního systému Windows Server. Cvičení se zaměřuje na základní instalaci, konfiguraci, správu uživatelů a skupin, a přehled klíčových rolí a funkcí Windows Serveru.

## 2. Použité nástroje a reference
*   **Windows Server:** [Homepage](https://www.microsoft.com/en-us/windows-server), [Dokumentace](https://docs.microsoft.com/en-us/windows-server/)
*   **Referenční materiál:** <http://home.zcu.cz/~pesicka/spos/WindowsCV2015.pdf> (Primární studijní materiál pro toto cvičení)

## 3. Poznámky ke cvičení (How-To)
_Vzhledem k tomu, že primární studijní materiál je externí PDF dokument, tato sekce poskytuje obecný přehled témat, která jsou typicky pokryta v rámci základů Windows Serveru. Pro detailní postupy a příkazy se prosím řiďte referenčním PDF dokumentem._

### Instalace a počáteční konfigurace
*   **Typy instalace:** Server Core vs. Desktop Experience.
*   **Základní síťová konfigurace:** IP adresa, DNS, brána.
*   **Správa serveru:** Použití Server Manageru.

### Správa uživatelů a skupin
*   **Lokální uživatelé a skupiny:** Vytváření, úpravy a správa lokálních uživatelských účtů a skupin.
*   **Oprávnění:** Nastavení oprávnění pro soubory a složky (NTFS oprávnění).

### Role a funkce Windows Serveru
*   **Instalace rolí a funkcí:** Použití Server Manageru nebo PowerShellu pro přidávání nových funkcionalit (např. Web Server (IIS), DHCP Server, DNS Server).
*   **Přehled základních rolí:** Krátký popis běžně používaných rolí.

### Základní příkazy a nástroje
*   **Příkazový řádek (CMD) a PowerShell:** Použití základních příkazů pro správu systému.
*   **Správce úloh (Task Manager):** Monitoring výkonu a procesů.
*   **Prohlížeč událostí (Event Viewer):** Kontrola systémových logů.

## 4. Příklad k procvičení

### Zadání

1.  Nainstalujte Windows Server (preferovaně s GUI – Desktop Experience) na virtuální stroj.
2.  Proveďte počáteční konfiguraci sítě (nastavte statickou IP adresu, masku, bránu a DNS server).
3.  Vytvořte dva nové lokální uživatelské účty: `uzivatel_a` a `uzivatel_b` s různými hesly.
4.  Vytvořte lokální skupinu `spravci_projektu` a přidejte do ní `uzivatel_a`.
5.  Vytvořte nový adresář `C:\ProjektData` a nastavte NTFS oprávnění tak, aby:
    *   Skupina `spravci_projektu` měla plná oprávnění.
    *   Uživatel `uzivatel_b` měl pouze oprávnění ke čtení.
    *   Ostatní uživatelé neměli žádný přístup.
6.  Nainstalujte roli "Web Server (IIS)" a ověřte její funkčnost přístupem přes webový prohlížeč.

### Ověření
*   Ověřte síťovou konfiguraci pomocí `ipconfig`.
*   Přihlaste se jako `uzivatel_a` a `uzivatel_b` a ověřte přístupová práva k `C:\ProjektData`.
*   Zkontrolujte, zda je webový server IIS dostupný na standardním HTTP portu 80.

## 5. Doplňující materiály
*   **Prezentace/Cvičení Windows Server:** <http://home.zcu.cz/~pesicka/spos/WindowsCV2015.pdf>
