# Automatizace a orchestrace (Ansible)

## 1. Úvod
**Cíl:** Seznámení se základy nástroje Ansible pro automatizaci správy konfigurace a nasazování aplikací. Cvičení zahrnuje pochopení struktury Ansible playbooků, inventáře, šablonování Jinja2 a základních úloh.

## 2. Použité nástroje a reference
*   **Ansible:** Nástroj pro automatizaci, správu konfigurace a nasazování aplikací.
    *   Homepage: [https://www.ansible.com/](https://www.ansible.com/)
    *   Dokumentace: [Ansible Documentation](https://docs.ansible.com/)
*   **Jinja2:** Šablonovací engine používaný Ansiblem.
    *   Homepage: [https://jinja.palletsprojects.com/](https://jinja.palletsprojects.com/)
    *   Dokumentace: [Jinja2 Documentation](https://jinja.palletsprojects.com/en/3.1.x/templates/)
*   **SSH (Secure Shell):** Protokol používaný Ansiblem pro komunikaci s managed nody.
    *   Reference: [man ssh](https://manpages.debian.org/ssh)
*   **YAML:** Jazyk pro serializaci dat používaný pro psaní Ansible playbooků a inventářů.
    *   Homepage: [https://yaml.org/](https://yaml.org/)
    *   Dokumentace: [YAML Specification](https://yaml.org/spec/1.2/spec.html)
*   **Vybrané Ansible moduly (příklady):**
    *   **group:** Správa skupin.
    *   **user:** Správa uživatelů.
    *   **template:** Šablonování souborů pomocí Jinja2.
    *   **file:** Správa souborů a adresářů.
    *   **hostname:** Správa názvu hostitele.
    *   **service:** Správa služeb.
    *   **ansible.builtin.shell:** Spouštění shell příkazů.
    *   *Reference:* Viz oficiální dokumentace Ansible pro konkrétní moduly.

## 3. Poznámky ke cvičení (How-To)

### Co je Ansible?
Ansible je open-source nástroj pro automatizaci, správu konfigurace a nasazování aplikací. Je bezagentový, což znamená, že nepotřebuje žádný software instalovaný na spravovaných uzlech – komunikuje s nimi přes SSH. Využívá jednoduchý YAML syntax pro definici úloh.

### Základní komponenty Ansible

#### Inventory (hosts.yml)
Inventory soubor definuje hosty (servery), které Ansible spravuje. Může být strukturován do skupin a podskupin. Zde je příklad inventáře:

```yaml
all:
  vars:
    remote_user: root
    ansible_ssh_user: root
    ansible_ssh_common_args: "-F ./ssh-config"
  children:
    spos:
      children:
        my-server:
          hosts:
            spos-01.kiv.zcu.cz:
```
*   `all`: Globální skupina pro všechny hosty.
*   `vars`: Proměnné platné pro celou skupinu nebo pro všechny hosty.
    *   `remote_user`, `ansible_ssh_user`: Uživatelské jméno pro SSH připojení.
    *   `ansible_ssh_common_args`: Dodatečné argumenty pro SSH (zde použití `ssh-config`).
*   `children`: Definuje podskupiny.
*   `hosts`: Seznam spravovaných hostů.

#### Playbook (playbook.yml)
Playbook je YAML soubor, který definuje sady úloh (tasks) pro spravované hosty.

```yaml
---
- name: Manage all hosts
  hosts: all
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
    vhost_port: 8081
    vhost_name: ansible.jindra.spos

  roles: []
  tasks:
    - name: Create 'sudo' group for sudo
      group:
        name: sudo
        state: present

    - name: Add/remove developers accounts to server
      user:
        name: "karel.novak"
        comment: "This is Karel"
        groups: sudo
        state: "present"
        shell: "/bin/bash"

    - name: Create new vhost for apache2
      template:
        src: vhost.j2
        dest: /etc/apache2/sites-enabled/{{ vhost_name }}
      notify: Restart apache2

    - name: Create document root
      file:
        path: /var/www/html/{{ vhost_name }}.{{ vhost_port }} 
        state: directory
        group: www-data
    
    - name: Create index.html
      template:
        src: index.j2
        dest: /var/www/html/{{ vhost_name }}.{{ vhost_port }}/index.html

    - name: Set the right hostname
      hostname:
        name: '{{ inventory_hostname }}'

    - name: This command will change the working directory to somedir/ and will only run when somedir/somelog.txt doesn't exist
      ansible.builtin.shell: touch somelog.txt
      args:
        chdir: /tmp/
        creates: somelog.txt

  handlers:
    - name: Restart apache2
      service:
        name: apache2
        state: restarted
```
*   `name`: Popis playbooku.
*   `hosts`: Na kterých hostech se má playbook spustit (zde na všech definovaných v inventáři).
*   `become: true`: Spustí úlohy s oprávněními roota (ekvivalent `sudo`).
*   `vars`: Lokální proměnné pro tento playbook.
*   `tasks`: Seznam úloh, které se mají provést. Každá úloha má `name` a používá modul Ansible (např. `group`, `user`, `template`, `file`, `hostname`, `ansible.builtin.shell`).
*   `handlers`: Akce, které se spustí pouze tehdy, pokud je na ně upozorněno (pomocí `notify`) a došlo ke změně.

#### SSH Configuration (ssh-config)
Tento soubor definuje nastavení SSH pro usnadnění připojení k hostům.

```
Host *
  ForwardAgent yes
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```
*   `Host *`: Platí pro všechny hosty.
*   `ForwardAgent yes`: Přeposílání SSH agenta.
*   `StrictHostKeyChecking no`: Vypíná kontrolu hostitelských klíčů (POZOR! Snižuje bezpečnost, vhodné jen pro testovací prostředí).
*   `UserKnownHostsFile /dev/null`: Nepoužívá `known_hosts` soubor.

#### Templates (index.j2, vhost.j2)
Ansible používá šablonovací systém Jinja2 pro generování konfiguračních souborů dynamicky na základě proměnných.

*   **index.j2:**
```
{{ vhost_name }}
```
*   **vhost.j2:**
```
<VirtualHost *:{{ vhost_port }}>
	ServerName {{ vhost_name }}

	ServerAdmin webmaster@{{ vhost_name }}
	DocumentRoot /var/www/html/{{ vhost_name}}.{{ vhost_port }}
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
Zde `{{ vhost_name }}` a `{{ vhost_port }}` jsou Jinja2 proměnné, které se nahradí hodnotami definovanými v playbooku.

## 4. Příklad k procvičení

### Zadání

1.  Upravte soubor `hosts.yml` tak, aby obsahoval alespoň dva servery ve skupině `webservers` a jeden server ve skupině `dbservers`.
2.  Rozšiřte `playbook.yml` o novou úlohu, která:
    *   Nainstaluje balíček `nginx` na servery ve skupině `webservers`.
    *   Zajistí spuštění služby `nginx`.
3.  Přidejte do playbooku úlohu, která vytvoří nového uživatele `backupuser` na všech serverech a zajistí mu SSH klíč (vygenerovaný lokálně) pro bezheslové přihlášení.

### Ověření
*   Spuštění playbooku:
    ```bash
    ansible-playbook -i hosts.yml playbook.yml
    ```
*   Přihlášení na spravované servery a ověření instalace `nginx` (`systemctl status nginx`), vytvoření uživatele `backupuser` a funkčnosti SSH klíče.

## 5. Doplňující materiály

### hosts.yml
```yaml
all:
  vars:
    remote_user: root
    ansible_ssh_user: root
    ansible_ssh_common_args: "-F ./ssh-config"
  children:
    spos:
      children:
        my-server:
          hosts:
            spos-01.kiv.zcu.cz:
```

### playbook.yml
```yaml
---
- name: Manage all hosts
  hosts: all
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
    vhost_port: 8081
    vhost_name: ansible.jindra.spos

  roles: []
  tasks:
    - name: Create 'sudo' group for sudo
      group:
        name: sudo
        state: present

    - name: Add/remove developers accounts to server
      user:
        name: "karel.novak"
        comment: "This is Karel"
        groups: sudo
        state: "present"
        shell: "/bin/bash"

    - name: Create new vhost for apache2
      template:
        src: vhost.j2
        dest: /etc/apache2/sites-enabled/{{ vhost_name }}
      notify: Restart apache2

    - name: Create document root
      file:
        path: /var/www/html/{{ vhost_name }}.{{ vhost_port }} 
        state: directory
        group: www-data
    
    - name: Create index.html
      template:
        src: index.j2
        dest: /var/www/html/{{ vhost_name }}.{{ vhost_port }}/index.html

    - name: Set the right hostname
      hostname:
        name: '{{ inventory_hostname }}'

    - name: This command will change the working directory to somedir/ and will only run when somedir/somelog.txt doesn't exist
      ansible.builtin.shell: touch somelog.txt
      args:
        chdir: /tmp/
        creates: somelog.txt

  handlers:
    - name: Restart apache2
      service:
        name: apache2
        state: restarted
```

### ssh-config
```
Host *
  ForwardAgent yes
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

### templates/index.j2
```
{{ vhost_name }}
```

### templates/vhost.j2
```
<VirtualHost *:{{ vhost_port }}>
	ServerName {{ vhost_name }}

	ServerAdmin webmaster@{{ vhost_name }}
	DocumentRoot /var/www/html/{{ vhost_name}}.{{ vhost_port }}
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```