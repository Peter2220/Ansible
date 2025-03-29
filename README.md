# Ansible Playbooks for Windows and Linux Automation

This repository contains a collection of Ansible Playbooks for automating tasks on both Windows and Linux environments. Ansible provides powerful automation tools that allow system administrators to manage servers and perform complex configuration management tasks in an easy and repeatable manner. These playbooks are designed to automate common administrative tasks and improve the efficiency of managing both Windows and Linux systems.

## Overview

- **Linux Automation**: These playbooks are designed to automate tasks commonly performed on Linux-based servers, such as package installation, configuration management, user management, and system monitoring.
- **Windows Automation**: These playbooks help automate tasks on Windows servers, including software installation, service management, user administration, and configuration tasks.

## Requirements

Before you begin using these playbooks, you must have the following prerequisites:

- Ansible 2.10+ installed on your machine.
- Python 3 installed.
- For Windows automation, you need `pywinrm` installed (can be done using `pip install pywinrm`).
- Access to a remote server (Linux or Windows) that you want to automate.

## Windows Automation

The Windows playbooks are designed to be run from a control machine (typically Linux or MacOS). To manage Windows hosts, you need to ensure that the following setup is complete:

1. Enable PowerShell Remoting on the Windows machine (`Enable-PSRemoting`).
2. Ensure the Windows machine is accessible via WinRM (Windows Remote Management).
3. Install the required Python libraries on your Ansible control machine: `pywinrm`.

### Example Playbook for Windows

This example demonstrates how to install the IIS web server on a Windows machine.

```yaml
---
- name: Install IIS on Windows
  hosts: windows
  gather_facts: yes
  tasks:
    - name: Install IIS
      win_feature:
        name: Web-Server
        state: present
