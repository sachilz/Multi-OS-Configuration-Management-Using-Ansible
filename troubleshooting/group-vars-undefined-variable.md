# Issue & Fix: Undefined Variable in Ansible Playbook

A simple troubleshooting guide explaining why Ansible failed with an `'ubuntu_ntp_package' is undefined` error and how moving/creating the `group_vars` folder resolved it.

---

## 1. The Error Encountered

When running:
```bash
ansible-playbook playbooks/configure-time-sync.yml
```

The execution failed with:
```text
TASK [Install Chrony on Ubuntu] ***********************************************************************************
fatal: [ubuntu-server]: FAILED! => {
    "msg": "The task includes an option with an undefined variable. The error was: 'ubuntu_ntp_package' is undefined. 'ubuntu_ntp_package' is undefined"
}
```

---

## 2. Root Cause Analysis

Even though `ubuntu_ntp_package` was defined inside `all.yml`, Ansible did **not** load the file.

### Why?
Ansible searches for `group_vars/` and `host_vars/` in only two specific locations:
1. **Next to the Playbook:** `<playbook_dir>/group_vars/` (e.g., `playbooks/group_vars/`)
2. **Next to the Inventory File:** `<inventory_dir>/group_vars/` (e.g., `inventory/group_vars/`)

### The Original Structure:
```text
project-root/
├── ansible.cfg
├── group_vars/          <-- SITTING AT ROOT (NOT SEARCHED AUTOMATICALLY)
│   └── all.yml
├── inventory/
│   └── hosts.ini        <-- Inventory is here
└── playbooks/
    └── configure-time-sync.yml  <-- Playbook is here
```

Because both the playbook (`playbooks/`) and inventory (`inventory/`) were in subdirectories, Ansible **did not inspect** the project root `./group_vars/all.yml`.

---

## 3. The Fix: Creating `inventory/group_vars/`

### The Solution:
Move or create the `group_vars/` folder inside the `inventory/` directory:

```bash
# In WSL / Linux terminal:
mv group_vars inventory/
```

### The Fixed Directory Structure:
```text
project-root/
├── ansible.cfg
├── inventory/
│   ├── hosts.ini
│   └── group_vars/      <-- NOW NEXT TO INVENTORY (AUTOMATICALLY LOADED!)
│       └── all.yml
├── learning/
├── playbooks/
│   └── configure-time-sync.yml
└── troubleshooting/
```

---

## 4. Alternative Fix: Explicit `vars_files`

If you ever prefer to keep variable files in a custom folder, you can explicitly tell the playbook to load them using `vars_files`:

```yaml
---
- name: Configure time synchronization across Linux distributions
  hosts: ubuntu
  become: true
  gather_facts: true
  vars_files:
    - ../group_vars/all.yml   # Explicit path to the variables file
```

---

## 5. Key Rule to Remember

> [!TIP]
> **Always place your `group_vars/` and `host_vars/` folders inside your `inventory/` directory.**
> This ensures that all playbooks using that inventory automatically inherit the correct variables without needing manual `vars_files` declarations.
