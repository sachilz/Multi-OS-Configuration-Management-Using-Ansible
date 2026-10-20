# SSH Key Setup Guide (WSL / Linux)

A simple guide on how to safely transfer, protect, and use an AWS EC2 SSH key (`.pem`) inside WSL for Ansible.

---

## 1. Summary of Commands

Here are the commands executed in order:

```bash
# 1. Create the .ssh directory if it doesn't exist
mkdir -p ~/.ssh

# 2. Copy the key from Windows Downloads to WSL ~/.ssh/
cp /mnt/c/Users/sachintha/Downloads/ansible-multios.pem ~/.ssh/

# 3. Set strict read-only permissions for the key
chmod 400 ~/.ssh/ansible-multios.pem

# 4. Verify file permissions
ls -l ~/.ssh/ansible-multios.pem
```

---

## 2. Command-by-Command Explanation

### Step 1: `mkdir -p ~/.ssh`
* **What it does:** Creates a hidden folder named `.ssh` in your Linux home directory (`~`).
* **Why `-p`:** The `-p` (parents) flag ensures no error is thrown if the directory already exists.

---

### Step 2: `cp /mnt/c/Users/.../Downloads/ansible-multios.pem ~/.ssh/`
* **What it does:** Copies the `.pem` file downloaded on Windows into your WSL Linux filesystem.
* **Understanding the path:**
  * In WSL, your Windows `C:` drive is mounted at `/mnt/c/`.
  * So your Windows Downloads folder is accessible at:
    ```
    /mnt/c/Users/<YourWindowsUser>/Downloads/
    ```

---

### Step 3: `chmod 400 ~/.ssh/ansible-multios.pem`
* **What it does:** Sets file permissions to **Read-only for the owner** (`-r--------`).
* **Why this is required:**
  * OpenSSH will **refuse** to use any private key that is readable by other users.
  * If permissions are not restricted, you will get the error:
    ```text
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0644 for '~/.ssh/...' are too open.
    ```
* **Permission breakdown (`400`):**
  * `4` (Owner): Read only
  * `0` (Group): No permissions
  * `0` (Others): No permissions

---

### Step 4: `ls -l ~/.ssh/ansible-multios.pem`
* **What it does:** Displays long listing details (permissions, owner, size, date) of the key.
* **Expected output:**
  ```text
  -r-------- 1 sachintha sachintha 1674 Oct 20 12:26 /home/sachintha/.ssh/ansible-multios.pem
  ```
  The leading `-r--------` confirms that only you have read permissions.

---

## 3. How to Use the Key

### Test Connection via SSH
Test direct SSH connection to your EC2 instance:
```bash
ssh -i ~/.ssh/ansible-multios.pem ubuntu@<EC2_PUBLIC_IP>
```

### Use in Ansible (`inventory/hosts.ini`)
Specify this key in your Ansible inventory file:
```ini
[ubuntu]
ubuntu-server ansible_host=<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible-multios.pem
```
