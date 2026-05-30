# Ansible Tags: `always`, `never`, and Standard Tags

This document summarizes how Ansible task tagging works, including the special tags `always` and `never`, standard tags, and untagged tasks.

---

# Complete Example Playbook

```yaml
---
- name: Tagging demo playbook
  hosts: all
  become: yes

  vars:
    pkg_name: httpd

  tasks:

    # ALWAYS runs
    - name: Pre-check - ensure OS is RedHat family
      assert:
        that: ansible_facts['os_family'] == "RedHat"
        fail_msg: "This playbook supports only RedHat systems"
      tags: always

    # ALWAYS runs
    - name: Gather minimal facts
      setup:
        gather_subset:
          - min
      tags: always

    # Standard tag
    - name: Install package
      yum:
        name: "{{ pkg_name }}"
        state: present
      tags: install

    # Standard tag
    - name: Start and enable service
      service:
        name: "{{ pkg_name }}"
        state: started
        enabled: yes
      tags: service

    # Multiple tags
    - name: Deploy config file
      copy:
        content: "Test config"
        dest: /tmp/test.conf
      tags:
        - config
        - deploy

    # NEVER task with additional tag
    - name: Restart service (manual operation)
      service:
        name: "{{ pkg_name }}"
        state: restarted
      tags:
        - never
        - restart

    # NEVER task (destructive)
    - name: Remove package
      yum:
        name: "{{ pkg_name }}"
        state: absent
      tags:
        - never
        - destructive

    # Optional debugging task
    - name: Debug system information
      debug:
        msg: "Hostname is {{ ansible_facts['hostname'] }}"
      tags:
        - never
        - debug

    # Untagged task
    - name: Create temp file
      file:
        path: /tmp/sample.txt
        state: touch
```

---

# Execution Behavior

## Run the Entire Playbook

```bash
ansible-playbook playbook.yml
```

Executed:

| Task Type             | Runs |
| --------------------- | ---- |
| always                | Yes  |
| Standard tagged tasks | Yes  |
| Untagged tasks        | Yes  |
| never tasks           | No   |

---

## Run Only Install Tasks

```bash
ansible-playbook playbook.yml --tags install
```

Executed:

| Task Type      | Runs |
| -------------- | ---- |
| always         | Yes  |
| install        | Yes  |
| Other tags     | No   |
| Untagged tasks | No   |
| never tasks    | No   |

---

## Run Deploy Tasks

```bash
ansible-playbook playbook.yml --tags deploy
```

Executed:

| Task Type          | Runs |
| ------------------ | ---- |
| always             | Yes  |
| deploy/config task | Yes  |
| Everything else    | No   |

---

## Run a Task Tagged with `never`

```bash
ansible-playbook playbook.yml --tags restart
```

Executed:

| Task Type    | Runs |
| ------------ | ---- |
| always       | Yes  |
| restart task | Yes  |

Although the task is tagged with `never`, it executes because the additional tag (`restart`) was explicitly requested.

---

## Run a Destructive Task

```bash
ansible-playbook playbook.yml --tags destructive
```

Executed:

| Task Type           | Runs |
| ------------------- | ---- |
| always              | Yes  |
| remove package task | Yes  |

This is a common pattern for dangerous operations that should only run when explicitly requested.

---

## Skip `always` Tasks

```bash
ansible-playbook playbook.yml --skip-tags always
```

Executed:

| Task Type            | Runs |
| -------------------- | ---- |
| always               | No   |
| Other selected tasks | Yes  |

Use carefully, as critical validation or fact-gathering tasks may be skipped.

---

# Common Tagging Patterns

## Pattern 1: Safe Manual Operations

```yaml
tags:
  - never
  - restart
```

Purpose:

* Excluded from normal execution.
* Runs only when explicitly requested.

Example:

```bash
ansible-playbook playbook.yml --tags restart
```

---

## Pattern 2: Mandatory Tasks

```yaml
tags: always
```

Purpose:

* Runs during normal execution.
* Runs even when specific tags are selected.

Example:

```bash
ansible-playbook playbook.yml --tags install
```

The `always` task still executes.

---

## Pattern 3: Untagged Tasks

```yaml
- name: Create temp file
  file:
    path: /tmp/sample.txt
    state: touch
```

Purpose:

* Runs only during full playbook execution.
* Does not run when using `--tags`.

---

# Mental Model

| Tag Type     | Behavior                                    |
| ------------ | ------------------------------------------- |
| always       | Force inclusion                             |
| never        | Force exclusion unless explicitly requested |
| Standard tag | Selective execution                         |
| No tag       | Runs only during full playbook execution    |

---

# Key Takeaways

* Use `always` for prerequisite checks, validation, and fact gathering.
* Use `never` for disruptive, risky, or manual operations.
* Use standard tags to organize functional areas of a playbook.
* Untagged tasks execute only when the entire playbook is run.
* Combining `never` with another tag is a common and recommended practice for administrative operations such as restarts, cleanup, or destructive actions.

---

# RHCE Exam Tips

1. Remember that `always` tasks run even when specific tags are selected.
2. `never` tasks do not run unless another matching tag is explicitly requested.
3. Untagged tasks are skipped when using `--tags`.
4. Combining `never` with descriptive tags is a best practice.
5. Read tag-related questions carefully; many exam scenarios depend on understanding exactly which tasks execute under different tag selections.
