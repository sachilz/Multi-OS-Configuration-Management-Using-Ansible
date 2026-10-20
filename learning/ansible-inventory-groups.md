# Ansible Inventory Groups & Multi-OS Ping

A beginner-friendly guide explaining parent-child groups in Ansible inventory and running commands against multiple OS platforms simultaneously.

---

## 1. The Command

```bash
ansible linux_servers -m ansible.builtin.ping
```

### What Happens:
Ansible reaches out to **all hosts** belonging to the `linux_servers` group (both Ubuntu and CentOS) simultaneously and tests connectivity.

### Example Output:
```json
ubuntu-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
centos-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

---

## 2. Understanding Parent-Child Groups (`:children`)

In your [inventory/hosts.ini](file:///c:/Users/sachintha/Desktop/Multi-OS-Configuration-Management-Using-Ansible/inventory/hosts.ini), you defined:

```ini
[ubuntu]
ubuntu-server ansible_host=18.208.215.0 ansible_user=ubuntu ansible_ssh_private_key_file=/home/sachintha/.ssh/ansible-multios.pem

[centos]
centos-server ansible_host=100.53.189.100 ansible_user=ec2-user ansible_ssh_private_key_file=/home/sachintha/.ssh/ansible-multios.pem

[linux_servers:children]
ubuntu
centos
```

### What does `:children` mean?
* `[linux_servers:children]` creates a **parent group** called `linux_servers`.
* Instead of listing individual hostnames or IPs under `linux_servers`, it lists other **groups** (`ubuntu` and `centos`).
* Any host that belongs to `ubuntu` or `centos` automatically belongs to `linux_servers`.

---

## 3. Why Group Hierarchies Are Crucial for Multi-OS Management

1. **Target Common Tasks to All Systems:**
   * When you want to ping, update common configurations, or push SSH keys, target the parent group:
     ```bash
     ansible linux_servers -m ansible.builtin.ping
     ```
2. **Target OS-Specific Tasks Separately:**
   * When package managers differ (`apt` on Ubuntu vs `dnf`/`yum` on CentOS/RHEL), target the specific child group:
     ```bash
     ansible ubuntu -m ansible.builtin.apt -a "name=curl state=present"
     ansible centos -m ansible.builtin.dnf -a "name=curl state=present"
     ```
3. **Multi-OS Playbook Design:**
   * You can write a single playbook with `hosts: linux_servers` and use `when: ansible_distribution == "Ubuntu"` or `when: ansible_distribution == "CentOS"` to handle differences cleanly.

---

## 4. Quick Reference Patterns

| Command | Target |
| :--- | :--- |
| `ansible ubuntu -m ping` | Only Ubuntu hosts |
| `ansible centos -m ping` | Only CentOS hosts |
| `ansible linux_servers -m ping` | All Ubuntu + CentOS hosts in the parent group |
| `ansible all -m ping` | Every host in the entire inventory |
