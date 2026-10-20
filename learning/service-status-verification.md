# Multi-OS Service Status Verification Using Ansible Ad-Hoc

A simple guide explaining how to verify service statuses (like Chrony) across both Ubuntu and CentOS servers using Ansible ad-hoc commands.

---

## 1. The Commands & Live Outputs

### On Ubuntu:
```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl is-active chrony"
```
**Output:**
```text
ubuntu-server | CHANGED | rc=0 >>
active
```

---

### On CentOS:
```bash
ansible centos -b -m ansible.builtin.command -a "systemctl is-active chronyd"
```
**Output:**
```text
centos-server | CHANGED | rc=0 >>
active
```

---

## 2. Command Breakdown

| Flag / Option | What It Does |
| :--- | :--- |
| **`ansible`** | CLI tool for executing one-off ad-hoc commands. |
| **`ubuntu` / `centos`** | The specific target group from [inventory/hosts.ini](file:///c:/Users/sachintha/Desktop/Multi-OS-Configuration-Management-Using-Ansible/inventory/hosts.ini). |
| **`-b`** | Short for `--become`. Runs the command with elevated `sudo`/root privileges. |
| **`-m ansible.builtin.command`** | The Ansible module that executes a binary/command on the target machine directly. |
| **`-a "systemctl is-active ..."`** | The argument passed to systemd to query whether the service is running. |

---

## 3. Key Multi-OS Difference: Service Naming

Notice a subtle difference between the two operating systems:

| OS Distribution | Package Name | Service Name in Systemd |
| :--- | :--- | :--- |
| **Ubuntu (Debian family)** | `chrony` | **`chrony`** |
| **CentOS (RHEL family)** | `chrony` | **`chronyd`** *(note the trailing 'd' for daemon)* |

> [!IMPORTANT]
> This is why multi-OS playbooks (like [playbooks/configure-time-sync.yml](file:///c:/Users/sachintha/Desktop/Multi-OS-Configuration-Management-Using-Ansible/playbooks/configure-time-sync.yml)) have separate tasks with `when: ansible_distribution == "Ubuntu"` and `when: ansible_distribution == "CentOS"`, or use variable mappings!

---

## 4. Understanding the Output

* **`rc=0`**: Return code 0 indicates success. For `systemctl is-active`, `0` means the service is actively running.
* **`active`**: Confirms that the NTP time sync daemon is running and synchronizing the system clock.
* **If the service was inactive:**
  * Output would say `inactive` or `failed` and return a non-zero exit code (`rc=3`).

---

## 5. Other Useful Systemd Ad-Hoc Checks

### Check full service status details:
```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl status chrony"
ansible centos -b -m ansible.builtin.command -a "systemctl status chronyd"
```

### Check if service starts automatically on system boot:
```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl is-enabled chrony"
ansible centos -b -m ansible.builtin.command -a "systemctl is-enabled chronyd"
```
