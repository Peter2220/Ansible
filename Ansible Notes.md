
# Ansible Learning Notes

> **NOTE:** Ansible study notes focused on Jinja2, Inventory, Roles, Collections, File Management, etc.
## Jinja2 Essentials

> **TIP:** Jinja2 appears everywhere in Ansible:
>
> * Templates
> * Variables
> * Conditions (`when`)
> * Loops
> * Inventory data
> * Facts

## Variables

```jinja2
{{ inventory_hostname }}

{{ ansible_facts['os_family'] }}

{{ ansible_default_ipv4.address }}
```
## Conditionals

```jinja2
{% if ansible_os_family == "RedHat" %}
yum install httpd
{% elif ansible_os_family == "Debian" %}
apt install apache2
{% endif %}
```

Inline conditional:

```jinja2
{{ "Enabled" if ansible_facts['distribution'] == "CentOS" else "Disabled" }}
```

## Loops

```jinja2
{% for user in users %}
User: {{ user }}
{% endfor %}
```

## Groups and Hostvars

### List Hosts

```jinja2
{% for host in groups['webservers'] %}
{{ host }}
{% endfor %}
```

### Generate `/etc/hosts`

```jinja2
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```

> **EXAM TIP:** Be comfortable using:
>
> * `groups`
> * `hostvars`
> * `ansible_facts`
> * Filters

### Useful Filters

| Filter  | Example                               |
| ------- | ------------------------------------- |
| default | `{{ var \| default('value') }}`       |
| upper   | `{{ hostname \| upper }}`             |
| join    | `{{ list \| join(',') }}`             |
| length  | `{{ list \| length }}`                |
| replace | `{{ hostname \| replace('-', '_') }}` |

---

# Inventory

## Useful Commands

```bash
ansible-navigator inventory -i inventory -m stdout --graph
```

Display specific group:

```bash
ansible-navigator inventory -i inventory -m stdout --graph production
```

> **TIP:** Use `--graph` frequently when troubleshooting inventory inheritance.

---

# Facts

## Gather Facts

```bash
ansible localhost -m setup --tree /tmp/facts
```

## View Facts

```bash
cat /tmp/facts/localhost
```

## Debug Facts

```yaml
- name: Show facts
  ansible.builtin.debug:
    var: ansible_facts
```

> **EXAM TIP:** Always know how to inspect facts before writing conditions.

---

# include_role vs import_role

## Quick Comparison

| Feature             | include_role | import_role |
| ------------------- | ------------ | ----------- |
| Type                | Dynamic      | Static      |
| Loops               | Yes          | No          |
| Runtime Parsing     | Yes          | No          |
| Variable Role Names | Yes          | No          |
| Performance         | Slower       | Faster      |

### include_role

```yaml
- ansible.builtin.include_role:
    name: myrole
```

> **TIP:** Use when role execution depends on runtime decisions.

### import_role

```yaml
- ansible.builtin.import_role:
    name: myrole
```

> **WARNING:** Cannot be used with loops.

```text
The 'loop' keyword cannot be used with 'import_role'
```

### Memory Aid

```text
include = execute later
import  = copy/paste now
```

---

# Roles

## Create Role

```bash
ansible-galaxy init myvhost
```

## Configure Roles Path

```ini
roles_path=roles
```

## Install Roles

```bash
ansible-galaxy role install -r roles/requirements.yml -p roles
```

> **EXAM TIP:** Understand role directory structure:
>
> * defaults/
> * vars/
> * tasks/
> * handlers/
> * templates/
> * files/

---

# Collections

## Install Collection

```bash
ansible-galaxy collection install community.crypto -p collections
```

## Install from Requirements

```bash
ansible-galaxy collection install -r collections/requirements.yml -p collections
```

> **IMPORTANT:** When using `ansible-navigator`, install collections inside the project directory.

Recommended:

```bash
ansible-galaxy collection install -p collections <collection>
```

### Why?

```text
~/.ansible/collections
```

may not be visible inside the execution-environment container.

---

# File Management

## lineinfile

```yaml
lineinfile:
  path: file.txt
  line: "example"
  state: present
  create: true
```

> **EXAM TIP:** One of the most frequently used modules in RHCE labs.

### Common Regex Pattern

```yaml
regexp: '^#?PasswordAuthentication'
```

## Symbolic Links

```yaml
state: link
force: true
```

> **WARNING:** Forgetting `force: true` is a common lab mistake.

## fetch

```text
Remote Host -> Control Node
```

### Memory Aid

```text
copy  = controller -> managed host
fetch = managed host -> controller
```

---

# Handlers

> **WARNING:** Handlers cannot be used inside:

```yaml
include_tasks
```

---

# SSH Configuration

### Enable Public Key Authentication

```yaml
regexp: '^#?PubkeyAuthentication'
line: 'PubkeyAuthentication yes'
```

### Enable Password Authentication

```yaml
regexp: '^#?PasswordAuthentication'
line: 'PasswordAuthentication yes'
```

### Restart Service

```yaml
- ansible.builtin.service:
    name: sshd
    state: restarted
```

---

# Common RHCE Pitfalls

> **WARNING:** Things that commonly cause lost points:

* Using `import_role` with loops
* Forgetting `force: true` for symlinks
* Using wrong inventory group names
* Forgetting handler notifications
* Installing collections into `~/.ansible/collections` when using `ansible-navigator`
* Forgetting `create: true` with `lineinfile`
* Confusing `defaults/` and `vars/`

---

# Highest Value Labs

> **EXAM TIP:** Practice these repeatedly.

| Priority | Lab                      |
| -------- | ------------------------ |
| High     | ch05s02 file-manage      |
| High     | ch05s04 file-template    |
| High     | ch06s05 projects-review  |
| High     | ch07s04 role-create      |
| High     | ch07s06 role-galaxy      |
| High     | ch07s08 role-collections |
| High     | review cr1               |
| High     | review cr2               |
| High     | review cr4               |
| High     | Q4 (SSH + lineinfile)    |
| High     | Q5 (find module)         |

---

# Useful Modules to Master

- lineinfile
- copy
- template
- find
- get_url
- service
- firewalld
- fetch
- include_role
- import_role
- debug
- setup
```
