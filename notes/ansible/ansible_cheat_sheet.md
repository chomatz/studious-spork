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

### configure logical volumes with conditions

```
---

- name: provision a logical volume with conditions
  hosts: <nodes>

  tasks:

    - block:

        - name: "create logical volume {{ lv_name }}"
          community.general.lvol:
            vg: "{{ vg_name }}"
            lv: "{{ lv_name  }}"
            size: 500

        - name: "format {{ lv_name }} with ext4"
          community.general.filesystem:
            dev: "/dev/{{ vg_name }}/{{ lv_name  }}"
            fstype: ext4

      rescue:

        - ansible.builtin.debug:
            msg: "failed to provision {{ lv_name }}"

        - name: retry with a smaller size
          community.general.lvol:
            vg: "{{ vg_name }}"
            lv: "{{ lv_name  }}"
            size: 200
      
        - name: "retry formatting {{ lv_name }} with ext4"
          community.general.filesystem:
            dev: "/dev/{{ vg_name }}/{{ lv_name  }}"
            fstype: ext4

      when: ansible_lvm.vgs.<vg_name> is defined

    - ansible.builtin.debug:
        msg: "volume group {{ ansible_lvm.vgs.<vg_name> }} does not exist"
      when: ansible_lvm.vgs.<vg_name> is not defined

...
```

### jinja2 templating

generate host file entries for all hosts

- jinja2 template

```
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
{% for host in ansible_play_hosts_all %}
{{ hostvars[host].ansible_facts.default_ipv4.address }} {{ hostvars[host].ansible_facts.fqdn }} {{ hostvars[host].ansible_facts.hostname }}
{% endfor %}
```

- playbook

```
---

- name: upload a dynamic template
  hosts: <nodes>

  tasks:

    - name: jinja 2 templating
      ansible.builtin.copy:
        dest: /etc/hosts
        src: </path/to/jinja2/template>
        owner: root
        group: root
        mode: "0644"

...
```

### create file(s)/dir(s) and set special permission

```
---

- name: create a directory
  ansible.builtin.file:
    path: /path/to/dir
    state: directory
    owner: <user>
    group: <group>
    mode: "0755"

- name: create a symlink
  ansible.builtin.file:
    path: /path/to/symlink
    src: /path/to/source
    state: link

- name: create a hardlink
  ansible.builtin.file:
    path: /path/to/hardlink
    src: /path/to/source
    state: hard

- name: set file permissions - set uid
  ansible.builtin.file:
    path: /path/to/file
    state: file
    mode: "04644"

- name: set file permissions - set gid
  ansible.builtin.file:
    path: /path/to/file
    state: file
    mode: "02644"

- name: set file permissions - set sticky bit
  ansible.builtin.file:
    path: /path/to/file
    state: file
    mode: "01644"

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

### initialization

```
ansible-galaxy role init --init-path </path/to/roles> <role_name>
```


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

### using roles

- sample 1

```
---

- name: sample 1 using role
  hosts: <nodes>

  roles:
    - <role_name1>
    - <role_name2>
    - <role_name3>

...
```

- sample 2

```
---

- name: sample 2 using include_role
  hosts: <nodes>

  tasks:

    - name: use include_role 1
      ansible.builtin.include_role:
        name: <role_name1>

    - name: use include_role 2
      ansible.builtin.include_role:
        name: <role_name2>

...
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
