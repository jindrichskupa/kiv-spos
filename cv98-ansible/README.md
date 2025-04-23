# Ansible


## Install

```bash
apt-get install -y ansible
```

## Configure

hosts.yml

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

.ssh/config


```
Host *
  ForwardAgent yes
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

playbook.yml

```yaml
---
- name: Manage all hosts
  hosts: all
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
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

    - name: Set the right hostname
      hostname:
        name: '{{ inventory_hostname }}'
```
