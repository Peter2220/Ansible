# Ansible Windows & Linux Inventory Setup

## Overview

This repository demonstrates how to configure Ansible to manage both:

* Linux (RHEL) hosts over SSH
* Windows hosts over WinRM

It also documents common issues and how to resolve them.

---

## Prerequisites

* Ansible installed
* Python `pip` available
* Network connectivity to target hosts

---

## Install Required Collections

Install the Windows collection:

```bash
ansible-galaxy collection install ansible.windows -p collections
```

---

## Install WinRM Python Dependency

Ansible requires `pywinrm` to communicate with Windows hosts:

You can verify what listeners are online by running this command on your Windows host
```cmd
winrm enumerate winrm/config/listener
```

```bash
pip install pywinrm
```

---

## Inventory Configuration

Example inventory file:

```ini
[win]
windows-host

[rhel]
linux-host

[rhel:vars]
ansible_user=ansible

[win:vars]
ansible_user=ansible
ansible_password=YourPassword
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_port=5985
ansible_winrm_server_cert_validation=ignore
ansible_shell_type=powershell
```

---

## Connectivity Tests

### Test Linux Hosts

```bash
ansible rhel -m ping
```

#### Possible Error

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
```

#### Cause

* SSH authentication not configured

#### Fix

* Configure SSH keys or enable password authentication

```bash
ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519):
Enter passphrase for "/root/.ssh/id_ed25519" (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub

ssh-copy-id ansible@linux-host

```


---

### Test Windows Hosts

```bash
ansible win -m ansible.windows.win_ping
```

---

## Common Issues and Fixes

### 1. Missing WinRM Library

#### Error

```text
winrm or requests is not installed: No module named 'winrm'
```

#### Fix

```bash
pip install pywinrm
```

---

### 2. WinRM Connection Timeout (Port 5986)

#### Error

```text
HTTPSConnectionPool ... port=5986 ... timed out
```

#### Cause

* WinRM HTTPS not configured
* Firewall blocking port 5986

#### Fix Options

**Option A: Use HTTP (simpler for lab environments)**

Ensure WinRM is enabled on the Windows host:

```powershell
winrm quickconfig
```

And use:

```ini
ansible_port=5985
```

---

### 3. Python Errors on Windows

#### Symptoms

* PowerShell errors referencing Python code
* Errors like:

```text
def _ansiballz_main():
```

#### Cause

* Ansible treating Windows host as Linux

#### Fix

* Ensure correct inventory group
* Ensure:

```ini
ansible_connection=winrm
ansible_shell_type=powershell
```

* Use Windows modules (`win_*`) instead of Linux modules

---

### 4. Wrong Module Usage

#### Incorrect

```yaml
- name: Run command
  shell: hostname
```

#### Correct

```yaml
- name: Run command
  ansible.windows.win_shell: hostname
```

---

## Verification

Check inventory parsing:

```bash
ansible-inventory --list
```

Ensure hosts appear under correct groups (`win`, `rhel`).

---

## Summary

| Issue                    | Cause                 | Fix                     |
| ------------------------ | --------------------- | ----------------------- |
| WinRM module missing     | pywinrm not installed | Install with pip        |
| Connection timeout       | Port/firewall issue   | Use 5985 or open 5986   |
| Python errors on Windows | Wrong connection type | Use WinRM + win modules |

---

## Notes

* Always use `ansible.windows` modules for Windows targets
* Avoid mixing Linux modules with Windows hosts
* Prefer HTTP (5985) for lab setups to simplify configuration
