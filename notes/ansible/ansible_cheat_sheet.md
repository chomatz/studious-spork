ansible cheat sheet
===================

ansible control node initial setup
----------------------------------

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

- create the `inventory` file

```
---
all:
  groupa:
    servera:
      ansible_host: <fqdn/ip>
    serverb:
      ansible_host: <fqdn/ip>
  groupb:
    serverc:
      ansible_host: <fqdn/ip>
    serverd:
      ansible_host: <fqdn/ip>
  groupc:
    children:
      groupa:
      groupb:
...
```

- check managed hosts

```
ansible-inventory --graph
ansible all -m ping
```

sample playbooks
----------------

### setup yum repositories

```
---

- name: configure yum repositories
  hosts: <hosta,hostb,groupa>

  tasks:

    - name: configure yum repository
      ansible.builtin.yum_repository:
        name: <repo_name>
        description: <repo_description>
        baseurl: <url>
        gpgcheck: true
        gpgkey: <key_location>
        enabled: true

...
```

### install packages

```
---

- name: install package set a
  hosts: <nodes>

  tasks:

    - name: install packagea and packageb
      ansible.builtin.package:
        name:
          - packagea
          - packageb
        state: present

- name: install package set b
  hosts: <nodes>

  tasks:

    - name: install packagec and packaged
      ansible.builtin.package:
        name:
          - packagec
          - packaged
        state: present

- name: update installed packages
  hosts: <nodes>

  tasks:

    - name: update packages
      ansible.builtin.package:
        name: "*"
        state: latest

...
```

linux-system-roles
------------------

- install linux-system-roles

```
sudo dnf install -y linux-system-roles
```

- individual role example(s) can be found at `/usr/share/doc/linux-system-roles/<role_name>`

collections
-----------

### requirements file

- sample `requirements.yml`

```
---

collections:
  - name: <collection_url1>
  - name: <collection_url2>
  - name: <collection_url3>

...
```

- install collection(s) using `requirements.yml`

```
ansible-galaxy collection install -r </path/to/requirements.yml>
```

roles
-----

### requirements file

- sample `requirements.yml`

```
---

- name: <role_name>
  src: <role_url>
- name: customrole
  src: https://fully.qualified.domain.name/customrole.tar

...
```

- install role(s) using `requirements.yml`

```
ansible-galaxy install -r </path/to/requirements.yml>
```

*user creation task
- name: create test user
  ansible.builtin.user:
    name: user
    comment: test user
    shell: /bin/bash
    groups: wheel
    password: "{{ secret | password_hash }}"

*special variables
- {{ ansible_fqdn }} # fully qualified domain name
- {{ ansible_default_ipv4.address }} # default ipv4 address
