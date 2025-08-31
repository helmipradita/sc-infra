# 🔐 SSH Key Management Guide

## 📋 Overview

Dokumentasi lengkap untuk mengelola SSH keys dalam infrastructure Ansible, termasuk troubleshooting common issues dan best practices.

## 🎯 Quick Reference

### Supported SSH Key Formats
| Format | Header | Compatible | Recommended |
|--------|--------|------------|-------------|
| **PEM/PKCS#1** | `-----BEGIN RSA PRIVATE KEY-----` | ✅ All tools | ✅ **YES** |
| **OpenSSH** | `-----BEGIN OPENSSH PRIVATE KEY-----` | ❌ Legacy tools | ❌ No |
| **PKCS#8** | `-----BEGIN PRIVATE KEY-----` | ⚠️ Some tools | ⚠️ Maybe |

### Key File Locations
```
ansible/keys/
├── aws-cloudhelmipradita.pem    # Main AWS key (PEM format)
├── monitoring-v2-clone2-custom.pem # Custom generated key (PEM format)
└── *.pem                        # Other project keys
```

## 🚨 Common Problems & Solutions

### Problem 1: "Load key: invalid format"

**Error:**
```
Load key "/path/to/key.pem": invalid format
Permission denied (publickey)
```

**Root Cause:**
- SSH key generated dengan OpenSSH format (default di Ubuntu 20.04+)
- Ansible/SSH client expect PEM format

**Quick Fix:**
```bash
# Convert existing key (if possible)
ssh-keygen -p -f key_file -m PEM -N ""

# OR generate new key with PEM format
ssh-keygen -t rsa -b 4096 -f ~/.ssh/custom-key -N "" -m PEM
```

**Prevention:**
Always use `-m PEM` flag when generating keys for Ansible.

### Problem 2: "Permission denied (publickey)"

**Scenarios:**

#### A. Wrong Key File
```bash
# Check if using correct key file
ansible-inventory -i inventory.ini --host hostname
```

#### B. Public Key Not Added to Server
```bash
# Extract public key from private key
ssh-keygen -y -f /path/to/private-key.pem

# Add to server's authorized_keys
echo "PUBLIC_KEY_CONTENT" >> ~/.ssh/authorized_keys
```

#### C. Wrong Permissions
```bash
# Fix private key permissions
chmod 600 /path/to/private-key.pem

# Fix server-side permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Problem 3: AWS Instance Without Default Key Pair

**Scenario:** Created AWS instance without selecting key pair during launch.

**Solution Options:**

#### Option A: Use AWS Console Browser Access
1. AWS Console → EC2 → Instance → Connect
2. Use "EC2 Instance Connect" 
3. Add your public key manually

#### Option B: Use AWS Session Manager
```bash
# Install AWS CLI and Session Manager plugin
aws ssm start-session --target i-INSTANCE_ID
```

#### Option C: Use Existing Key from Other Servers
Most efficient - reuse proven working keys across infrastructure.

## 📋 Step-by-Step Guides

### Guide 1: Setup Custom SSH Key for New Server

**Prerequisites:**
- Server accessible via AWS Console or existing SSH
- Local Ansible project setup

**Steps:**

#### Step 1: Generate SSH Key (Correct Format)
```bash
# On server or local machine
ssh-keygen -t rsa -b 4096 -f ~/.ssh/project-custom-key -N "" -m PEM
#                                                              ^^^^
#                                                              Force PEM format
```

#### Step 2: Copy Keys to Correct Locations
```bash
# Copy private key to project
cp ~/.ssh/project-custom-key /path/to/project/ansible/keys/custom-key.pem
chmod 600 /path/to/project/ansible/keys/custom-key.pem

# Add public key to server
cat ~/.ssh/project-custom-key.pub >> ~/.ssh/authorized_keys
```

#### Step 3: Update Inventory
```ini
[all_servers]
new-server ansible_host=IP ansible_user=ubuntu key_file=custom-key.pem
```

#### Step 4: Test Connection
```bash
ansible -i ansible/inventories/inventory.ini new-server -m ping
```

### Guide 2: Reuse Existing Key Across Servers

**Most Recommended Approach**

#### Step 1: Extract Public Key from Existing Private Key
```bash
ssh-keygen -y -f ansible/keys/aws-cloudhelmipradita.pem
```

#### Step 2: Add Public Key to New Server
```bash
# Via AWS Console Browser or existing SSH connection
echo "EXTRACTED_PUBLIC_KEY" >> ~/.ssh/authorized_keys
```

#### Step 3: Use Same Key in Inventory
```ini
[all_servers]
new-server ansible_host=IP ansible_user=ubuntu key_file=aws-cloudhelmipradita.pem
```

### Guide 3: Fix Invalid Format Error

**When you get "invalid format" error:**

#### Diagnosis:
```bash
# Check key format
head -1 /path/to/key.pem

# If shows: -----BEGIN OPENSSH PRIVATE KEY-----
# Then it needs conversion
```

#### Fix Method 1: Convert Existing Key
```bash
# Try to convert (may not work if format too different)
ssh-keygen -p -f key_file -m PEM -N ""
```

#### Fix Method 2: Generate New Key with Correct Format
```bash
# Generate new key
ssh-keygen -t rsa -b 4096 -f new-key -N "" -m PEM

# Replace old key
mv new-key /path/to/ansible/keys/fixed-key.pem
chmod 600 /path/to/ansible/keys/fixed-key.pem

# Update server authorized_keys with new public key
ssh-keygen -y -f /path/to/ansible/keys/fixed-key.pem >> ~/.ssh/authorized_keys
```

## 🛡️ Security Best Practices

### File Permissions
```bash
# Private keys
chmod 600 ansible/keys/*.pem

# SSH directory on servers  
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Key Management
- **Never commit private keys** to git
- Use `.gitignore` for `ansible/keys/` directory
- Rotate keys periodically
- Use different keys for different environments (dev/staging/prod)

### Access Control
```bash
# Inventory file controls which key is used for which server
[all_servers]
prod-server   key_file=prod-key.pem
stage-server  key_file=stage-key.pem  
dev-server    key_file=dev-key.pem
```

## 🧪 Testing & Validation

### Test Individual Server Connection
```bash
# Test specific server
ansible -i ansible/inventories/inventory.ini server-name -m ping

# Test with verbose output for debugging
ansible -i ansible/inventories/inventory.ini server-name -m ping -vvv
```

### Test All Servers
```bash
# Test all servers in inventory
ansible -i ansible/inventories/inventory.ini all_servers -m ping

# Show success/failure summary
echo $?  # 0 = all success, non-zero = some failed
```

### Manual SSH Test
```bash
# Test SSH connection manually
ssh -i ansible/keys/key-file.pem user@server-ip

# Test with verbose SSH output
ssh -vvv -i ansible/keys/key-file.pem user@server-ip
```

## 🔧 Advanced Troubleshooting

### Debug Ansible SSH Issues
```bash
# Show all variables for a host
ansible-inventory -i inventory.ini --host hostname

# Test with maximum verbosity
ansible -i inventory.ini hostname -m ping -vvvv

# Check SSH key loading
ssh-add -l  # List loaded keys
ssh-add ansible/keys/key-file.pem  # Add key to agent
```

### Server-Side Debugging
```bash
# Check SSH logs on server
sudo tail -f /var/log/auth.log

# Check authorized_keys content
cat ~/.ssh/authorized_keys

# Test key formats
ssh-keygen -l -f ~/.ssh/authorized_keys
```

### Network/Firewall Issues
```bash
# Test port 22 connectivity
telnet server-ip 22

# Check security groups (AWS)
# Ensure port 22 is open for your IP

# Check SSH service status on server
sudo systemctl status ssh
```

## 📊 Troubleshooting Decision Tree

```
SSH Connection Failed?
├── Error: "invalid format"
│   ├── Check key format (PEM vs OpenSSH)
│   ├── Convert key: ssh-keygen -p -f key -m PEM
│   └── Or generate new PEM key
├── Error: "permission denied"
│   ├── Check key file permissions (600)
│   ├── Verify public key in authorized_keys
│   ├── Check server SSH permissions
│   └── Test manual SSH connection
├── Error: "connection refused"
│   ├── Check server is running
│   ├── Check port 22 accessibility
│   ├── Check security groups/firewall
│   └── Verify SSH service status
└── Error: "host unreachable"
    ├── Check IP address in inventory
    ├── Check network connectivity
    └── Verify server is running
```

## 🎯 Real-World Examples

### Example 1: Standard AWS Setup
```ini
# inventory.ini
[all_servers]
web-server    ansible_host=1.2.3.4   ansible_user=ubuntu   key_file=aws-main.pem
db-server     ansible_host=1.2.3.5   ansible_user=ubuntu   key_file=aws-main.pem
```

### Example 2: Mixed Environment Setup
```ini
# inventory.ini
[all_servers]
ubuntu-server ansible_host=1.2.3.4   ansible_user=ubuntu   key_file=ubuntu-key.pem
centos-server ansible_host=1.2.3.5   ansible_user=centos   key_file=centos-key.pem
custom-server ansible_host=1.2.3.6   ansible_user=ubuntu   key_file=custom-key.pem
```

### Example 3: Our Current Setup
```ini
[all_servers]
monitoring-v2      ansible_host=13.212.82.138  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
monitoring-v2-clone1 ansible_host=47.128.252.103 ansible_user=ubuntu key_file=aws-cloudhelmipradita.pem
monitoring-v2-clone2-without-default-key ansible_host=54.179.57.187 ansible_user=ubuntu key_file=monitoring-v2-clone2-custom.pem
```

## 📚 Additional Resources

### Useful Commands Cheat Sheet
```bash
# Generate PEM format key
ssh-keygen -t rsa -b 4096 -f keyname -N "" -m PEM

# Extract public key from private
ssh-keygen -y -f private-key.pem

# Test key format
file private-key.pem

# Check key fingerprint
ssh-keygen -l -f private-key.pem

# Add key to SSH agent
ssh-add private-key.pem

# Test SSH connection with specific key
ssh -i private-key.pem user@host
```

### Key Format Identification
```bash
# PEM format (GOOD for Ansible)
-----BEGIN RSA PRIVATE KEY-----

# OpenSSH format (May cause issues)  
-----BEGIN OPENSSH PRIVATE KEY-----

# PKCS#8 format (Usually works)
-----BEGIN PRIVATE KEY-----
```

---

**⚠️ Important Notes:**
- Always backup working keys before making changes
- Test SSH connection before running Ansible playbooks
- Keep private keys secure and never commit to version control
- Document which keys are used for which servers