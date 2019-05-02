# Cviceni 3 - vystup

## Zadani

1. Nainstalujte bind9 a nastavte slave zonu spos-test.spos z 192.168.255.42
2. Vytvorte vlastni zonu \<orion-login\>.spos-test.spos a dejte ji public, povolte zonovy transfer z 192.168.255.0/24
3. Vytvorte reverzni zaznam pro 192.168.255.\<vase ip\> na \<orion-login\>.spos-test.spos
4. Ve vytvorene zone nastavte nasledujici zaznamy:
  * prijem posty na stejne servery jako spos-test.spos
  * TXT zaznam pro SPF (<http://www.spfwizard.net/>) pro mailservery, servery s Ackovym zaznamem na hlavni domenu a servery s teverzem v dane zone
  * alias www.\<orion-login\>.spos-test.spos na server.\<orion-login\>.spos-test.spos
  * preklad server.\<orion-login\>.spos-test.spos 192.168.255.\<vase ip\>
5. Vytvorte script ktery nastaveni overi

## Kontrola

Spusteni scriptu, ktery overi nastaveni (ruzne kombinace `host` / `dig`)