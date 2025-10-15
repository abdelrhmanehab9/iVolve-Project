# Ansible Dynamic Inventory with AWS EC2

## Overview
This guide demonstrates how to set up Ansible dynamic inventory to automatically discover and manage running EC2 instances on AWS, eliminating the need for manual inventory file management.

---

## Prerequisites

- AWS Account with access credentials
- Ansible installed on your control machine
- AWS CLI installed and configured
- Basic understanding of Ansible and AWS EC2

---

## Step 1: Launch EC2 Instance on AWS

1. Navigate to **AWS Console** → **EC2** → **Launch Instance**
2. Configure the instance:
   - **Name Tag**: `ivolve`
   - **Instance Type**: `t2.micro` (or any preferred type)
   - **AMI**: Amazon Linux 2 or Ubuntu
   - **Key Pair**: Create or select existing key pair (download `.pem` file)
   - **Security Group**: Allow inbound SSH (port 22)
3. Launch the instance and wait until it's in **running** state
4. Note the **Public IP** address (optional for verification)

---

## Step 2: Install Required Ansible Collection

Install the AWS collection for Ansible:

```bash
ansible-galaxy collection install amazon.aws
```

Install boto3 (required for AWS integration):

```bash
pip3 install boto3 botocore
```

---

## Step 3: Configure AWS Credentials

```bash
aws configure
```

```
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
Default region name: us-east-1
Default output format: json
```

This saves credentials to `~/.aws/credentials`

---

## Step 4: Create Dynamic Inventory Configuration

```bash
cd ~/ansible-lab
mkdir -p inventory
cd inventory
nano aws_ec2.yml
```

```yaml
plugin: amazon.aws.ec2
regions:
  - us-east-1  # Change to your EC2 region

keyed_groups:
  - key: tags.Name
    prefix: tag

filters:
  tag:Name: ivolve
  instance-state-name: running

compose:
  ansible_host: public_ip_address
```

**Configuration Explanation:**
- `plugin`: Specifies the AWS EC2 plugin
- `regions`: AWS region(s) to query
- `keyed_groups`: Groups instances by Name tag
- `filters`: Only includes instances with Name=ivolve and running state
- `compose`: Uses public IP as ansible_host

---

## Step 5: Verify Connectivity

### Test with Ad-hoc Command

Run a ping test:

```bash
ansible -i inventory/aws_ec2.yml all -m ping --private-key ~/.ssh/your-key.pem -u ec2-user
```

---

## Step 6: Create Ansible Configuration File (Optional)

To avoid repeating parameters, create an `ansible.cfg` file:

```bash
nano ansible.cfg
```

Add:

```ini
[defaults]
inventory = inventory/aws_ec2.yml
private_key_file = ~/.ssh/your-key.pem
remote_user = ec2-user
host_key_checking = False

[inventory]
enable_plugins = amazon.aws.ec2
```

---

## Common Use Cases

### Filter by Multiple Tags

```yaml
filters:
  tag:Environment: production
  tag:Project: ivolve
  instance-state-name: running
```

### Group by Instance Type

```yaml
keyed_groups:
  - key: instance_type
    prefix: type
```

### Use Private IP Instead

```yaml
compose:
  ansible_host: private_ip_address
```

---

## Conclusion

I have successfully configured Ansible dynamic inventory with AWS EC2. The inventory will now automatically discover and manage my EC2 instances without manual updates.

**Key Benefits:**
- Automatic instance discovery
- No manual inventory management
- Real-time instance information
- Scalable infrastructure management
