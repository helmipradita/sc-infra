# 📋 Inventory Management Guide

## Quick Start

Inventory file location: `ansible/inventory.ini`

### 🎯 Most Common Tasks

#### 1. Add New Server
Edit `[all_servers]` section:
```ini
[all_servers]
ubuntu-server ansible_host=13.228.129.190 ansible_user=ubuntu   key_file=aws-cloudhelmipradita.pem
aws-linux     ansible_host=13.251.120.59  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
new-server    ansible_host=NEW_IP_HERE     ansible_user=ubuntu   key_file=aws-cloudhelmipradita.pem
```

#### 2. Install Services on Server
Move server to appropriate service group:

**For web services (nginx):**
```ini
[web]
ubuntu-server
new-server    # <- Add here for nginx
```

**For monitoring (grafana + prometheus):**
```ini
[monitoring]
ubuntu-server
new-server    # <- Add here for monitoring stack
```

**For metrics collection (node_exporter):**
```ini
[metrics]
ubuntu-server
aws-linux
new-server    # <- Add here for metrics
```

## 📖 Detailed Guide

### Server Types

#### Ubuntu Servers
```ini
server-name ansible_host=IP ansible_user=ubuntu key_file=key-name.pem
```

#### Amazon Linux Servers
```ini
server-name ansible_host=IP ansible_user=ec2-user key_file=key-name.pem
```

### Service Groups

| Group | Services Installed | Purpose |
|-------|-------------------|---------|
| `[web]` | nginx | Web server, reverse proxy |
| `[monitoring]` | grafana + prometheus | Monitoring dashboard + metrics collector |
| `[metrics]` | node_exporter | System metrics collection |

### Key Files

All SSH keys should be placed in: `ansible/keys/`

**Current Keys:**
- `aws-cloudhelmipradita.pem` - Main AWS key (PEM format) ✅ Working
- `monitoring-v2-clone2-custom.pem` - Custom generated key (PEM format) ✅ Working

**⚠️ SSH Key Format Requirements:**
- Must be **PEM format**: `-----BEGIN RSA PRIVATE KEY-----`
- **NOT** OpenSSH format: `-----BEGIN OPENSSH PRIVATE KEY-----`
- Generate with: `ssh-keygen -t rsa -b 4096 -f keyname -N "" -m PEM`

**📋 For detailed SSH key troubleshooting, see [SSH-KEY-MANAGEMENT.md](./SSH-KEY-MANAGEMENT.md)**

## 🔧 Examples

### Example 1: Add Production Web Server
```ini
# 1. Add to servers
[all_servers]
ubuntu-server ansible_host=13.228.129.190 ansible_user=ubuntu   key_file=aws-cloudhelmipradita.pem
aws-linux     ansible_host=13.251.120.59  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
prod-web      ansible_host=10.0.1.100     ansible_user=ubuntu   key_file=prod-key.pem

# 2. Install nginx + metrics
[web]
ubuntu-server
prod-web

[metrics]
ubuntu-server
aws-linux
prod-web
```

### Example 2: Add Monitoring Server
```ini
# 1. Add to servers
[all_servers]
ubuntu-server ansible_host=13.228.129.190 ansible_user=ubuntu   key_file=aws-cloudhelmipradita.pem
aws-linux     ansible_host=13.251.120.59  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
monitor-srv   ansible_host=10.0.1.200     ansible_user=ubuntu   key_file=monitor-key.pem

# 2. Install monitoring stack
[monitoring]
ubuntu-server
monitor-srv

[metrics]
ubuntu-server
aws-linux
monitor-srv
```

### Example 3: Add Multiple AWS Instances
```ini
# 1. Add to servers
[all_servers]
ubuntu-server ansible_host=13.228.129.190 ansible_user=ubuntu   key_file=aws-cloudhelmipradita.pem
aws-linux     ansible_host=13.251.120.59  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
aws-web1      ansible_host=13.250.100.10  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem
aws-web2      ansible_host=13.250.100.11  ansible_user=ec2-user key_file=aws-cloudhelmipradita.pem

# 2. Deploy web servers with metrics
[web]
ubuntu-server
aws-web1
aws-web2

[metrics]
ubuntu-server
aws-linux
aws-web1
aws-web2
```

## ⚠️ Important Notes

### 1. Always Update Both Sections
When adding servers, update:
1. `[all_servers]` - basic connection info
2. Service groups (`[web]`, `[monitoring]`, `[metrics]`) - what to install

### 2. Key File Naming
- Key files go in `ansible/keys/`
- Reference only filename in `key_file=` parameter
- Example: `key_file=my-key.pem` → file at `ansible/keys/my-key.pem`

### 3. User Names by OS
- **Ubuntu**: `ansible_user=ubuntu`
- **Amazon Linux**: `ansible_user=ec2-user`
- **CentOS/RHEL**: `ansible_user=centos` or `ansible_user=ec2-user`

### 4. Testing Changes
After editing inventory, test with:
```bash
# Test connection
ansible -i ansible/inventory.ini all_servers -m ping

# List all servers and their variables
ansible-inventory -i ansible/inventory.ini --list --yaml
```

## 🚫 What NOT to Edit

**RARELY EDITED sections** (bottom of inventory.ini):
- `[ubuntu]` / `[amazon_linux]` - Auto-managed
- `[web:vars]` / `[monitoring:vars]` / `[metrics:vars]` - Service definitions
- `[all:vars]` - Connection settings
- `[all_servers:vars]` - Default values

Only modify these if you need to:
- Change SSH settings
- Add new service types
- Modify Python interpreters

## 🆘 Troubleshooting

### Connection Issues
```bash
# Test specific server
ansible -i ansible/inventory.ini server-name -m ping

# Debug connection
ansible -i ansible/inventory.ini server-name -m ping -vvv
```

### Variable Issues
```bash
# Check what variables a server has
ansible-inventory -i ansible/inventory.ini --host server-name
```

### Service Not Installing
1. Check server is in correct service group
2. Verify service variables are set correctly
3. Test with specific playbook:
```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-nginx.yml --limit server-name
```