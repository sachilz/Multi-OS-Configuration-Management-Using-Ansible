# Service Status Verification Using Ansible Ad-Hoc

A simple guide explaining how to verify whether a service (like Chrony) is active on remote hosts using an Ansible ad-hoc command.

---

## 1. The Command

```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl is-active chrony"
```

### Example Output:
```text
ubuntu-server | CHANGED | rc=0 >>
active
```

---

## 2. Command Breakdown

| Flag / Argument | What It Does |
| :--- | :--- |
| **`ansible`** | CLI tool for executing one-off tasks across managed nodes. |
| **`ubuntu`** | Target group defined in your [inventory/hosts.ini](file:///c:/Users/sachintha/Desktop/Multi-OS-Configuration-Management-Using-Ansible/inventory/hosts.ini). |
| **`-b`** | Short for `--become`. Runs the command with elevated/root privileges (`sudo`). |
| **`-m ansible.builtin.command`** | Uses Ansible's built-in `command` module to execute a binary on the remote host without going through a shell. |
| **`-a "systemctl is-active chrony"`** | The arguments passed to the command module. Tells systemd to report the running state of the `chrony` service. |

---

## 3. Why Use `-b` (`--become`)?

Some services and system inspections require root permissions to query or manipulate system states. 
Adding `-b` ensures that Ansible elevates privileges using `sudo`, preventing "permission denied" errors when interacting with system daemons.

---

## 4. Understanding the Output

* **`rc=0`**: Return code 0 means the command succeeded. In `systemctl is-active`, return code `0` means the service is **up and running**.
* **`active`**: The textual status returned by `systemctl`.
* **If the service was stopped:**
  * Output would show `inactive` or `failed` with a non-zero return code (e.g., `rc=3`).

---

## 5. Equivalent Commands for Other Linux Distros / Services

You can adapt this command for other services across your multi-OS environment:

### Check Chrony on CentOS/RHEL (`chronyd`):
```bash
ansible centos -b -m ansible.builtin.command -a "systemctl is-active chronyd"
```

### Check service status with full details (`systemctl status`):
```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl status chrony"
```

### Check if service is enabled on boot:
```bash
ansible ubuntu -b -m ansible.builtin.command -a "systemctl is-enabled chrony"
```
