
---

```markdown
# Using Ansible Variables — Examples

This directory contains basic examples demonstrating how to use variables in Ansible playbooks, including static variable assignments and looping through lists defined in external variable files.

## Files

### `myvars.yaml`
This is a **variable file** used to define both individual and list-based variables. It includes:
- A list of package names (`packages`) to be used in loops.
- Individual variables like `mypackage`, `myftpservice`, and `myfileservice`.

```yaml
packages:
  - nmap
  - vsftpd
  - cockpit
mypackage: nmap
myftpservice: vsftpd
myfileservice: smb
```

---

### `variables_example.yaml`
This playbook demonstrates how to use **inline variables** within a play. It defines a `user` variable and creates a user account on the target host.

```yaml
- name: create a user using a variable
  hosts: all
  vars:
    user: Ramos
  tasks:
    - name: create a user {{ user }} on host {{ ansible_hostname }}
      user:
        name: "{{ user }}"
```

---

### `vars_file.yaml`
This playbook demonstrates how to use **external variable files** and **loops** with Ansible.

It loads variables from `myvars.yaml`, installs each package listed in the `packages` array, and then enables and starts the corresponding services.

```yaml
- name: using a variable include file
  hosts: rhel
  vars_files: myvars.yaml
  tasks:
    - name: install package
      yum:
        name: "{{ item }}"
        state: latest
      loop: "{{ packages }}"

    - name: start and enable service
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop: "{{ packages }}"
```

---

## Usage

To run these playbooks, use the `ansible-playbook` command:

```bash
ansible-playbook variables_example.yaml
ansible-playbook vars_file.yaml
```

Make sure you have the appropriate inventory (`-i inventory_file`) and permissions set up for the target hosts.

---
```
