# AWS CLI Configuration Guide

A simple, beginner-friendly guide on how to configure AWS credentials using named profiles and check resources.

---

## 1. What Command Was Used?

```bash
aws configure --profile ansible-project
```

### Why use `--profile`?
By default, `aws configure` sets credentials for the `default` profile. Using `--profile <name>` allows you to manage multiple AWS accounts or project-specific credentials without overwriting each other.

---

## 2. Prompts Explained

When you run `aws configure`, it asks for 4 inputs:

| Prompt | Description | Example |
| :--- | :--- | :--- |
| **AWS Access Key ID** | Public identifier for your AWS IAM user | `YOUR_ACCESS_KEY_ID` |
| **AWS Secret Access Key** | Secret password/key paired with your Access Key | `YOUR_SECRET_ACCESS_KEY` |
| **Default region name** | AWS region where your resources will be provisioned | `us-east-1` |
| **Default output format** | Format in which CLI responses are shown (`json`, `yaml`, `text`, or `table`) | `json` |

> [!WARNING]
> **Never commit your actual Access Key ID or Secret Access Key to Git or public repositories!** Always keep them private.

---

## 3. Where Are These Credentials Stored?

AWS CLI saves configuration in hidden files under your user's home directory (`~/.aws/`):

1. **`~/.aws/credentials`**: Stores your Access Key ID and Secret Access Key.
2. **`~/.aws/config`**: Stores the default region and output format.

---

## 4. Important Security Warning & Fix

If you see this warning:
```text
aws: [WARNING]: The file '/home/<user>/.aws/credentials' is accessible by other users.
```

It means other users on your machine have permission to read your secret keys. 

### Fix:
Restrict permissions so only you (the owner) can read and write the credentials file:

```bash
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config
```

---

## 5. Verify Your Configuration

You can quickly check if your profile is properly configured:

```bash
aws configure list --profile ansible-project
```

This displays a table showing where the profile, access keys, and region are being loaded from.

---

## 6. Listing and Inspecting EC2 Instances

To list and view details of your EC2 instances (such as Public IP, State, Instance ID) for Ansible:

```bash
aws ec2 describe-instances --profile ansible-project --region us-east-1
```

### What does this command do?
- **`ec2 describe-instances`**: Retrieves detailed metadata about all EC2 instances in your account.
- **`--profile ansible-project`**: Uses the specific credentials configured for this project.
- **`--region us-east-1`**: Queries instances located in the North Virginia (`us-east-1`) region.

### Quick Tips for Ansible Users:
1. **Find Public IPs quickly (using `--query`):**
   ```bash
   aws ec2 describe-instances --profile ansible-project --region us-east-1 \
     --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress,State.Name]" \
     --output table
   ```
2. **Filter only running instances:**
   ```bash
   aws ec2 describe-instances --profile ansible-project --region us-east-1 \
     --filters "Name=instance-state-name,Values=running"
   ```

---

## 7. How to Use the Configured Profile

Whenever you run AWS CLI commands or Ansible playbooks, you can specify this profile:

### Option A: Use the `--profile` flag in commands
```bash
aws sts get-caller-identity --profile ansible-project
```

### Option B: Set as environment variable for the session
```bash
export AWS_PROFILE=ansible-project
aws sts get-caller-identity
```

### Option C: In Ansible
In your Ansible playbooks or `hosts.ini`/vars, you can specify the AWS profile:
```yaml
aws_profile: "ansible-project"
```
