ansible cheat sheet
===================

ansible control node setup
--------------------------

- install ansible-navigator

```
sudo dnf install -y ansible-core ansible-navigator
```

- login to image registry and pull required images

```
podman login <registry_url>
podman pull <registry_url/image_name:image_tag>
```

- create `ansible.cfg`

```
[defaults]
inventory = </path/to/inventory/file/or/directory>
collections_path: </path/to/collections/directory>
roles_path: </path/to/roles/directory>
remote_user = <ssh_user_name>
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```

- create `ansible-navigator.yml`

```
ansible-navigator:
  execution-environment:
    image: registry_url/image_name:image_tag
    pull:
      policy: missing
  playbook-artifact:
    enable: false
```

*yml inventory
---
all:
  hosts:
    servera:
      ansible_host: <fqdn/ip>
    serverb:
      ansible_host: <fqdn/ip>
  vars:
    ansible_user: <username>
    ansible_ssh_private_key_file: </path/to/ssh/private/key>
    ansible_become: yes
    ansible_become_method: sudo
    ansible_become_pass: <password>
...

*collection installation
ansible-galaxy collection install </path/to/archive> -p collections
ansible-galaxy collection install <collection_name> -p collections

*user creation task
- name: create test user
  ansible.builtin.user:
    name: examinee
    comment: test user
    shell: /bin/bash
    groups: wheel
    password: "{{ secret | password_hash }}"

*special variables
- {{ ansible_fqdn }} # fully qualified domain name
- {{ ansible_default_ipv4.address }} # default ipv4 address
