# 🔧 Ansible Directory Structure

## 📁 Directory Overview

```
ansible/
├── inventories/           # Inventory configurations
│   └── inventory.ini      # Main inventory file (edit frequently)
├── configs/              # Ansible configuration files
│   └── ansible.cfg       # Ansible settings
├── playbooks/            # Organized playbooks
│   ├── install/          # Individual service installations
│   ├── deploy/           # Deployment orchestration
│   ├── setup/            # Infrastructure setup
│   ├── utils/            # Utilities and maintenance
│   └── README.md         # Playbooks documentation
├── keys/                 # SSH private keys
│   └── *.pem            # AWS/server access keys
├── group_vars/           # Group-specific variables
├── host_vars/            # Host-specific variables  
├── roles/                # Ansible roles (if used)
├── templates/            # Jinja2 templates
│   ├── docker-compose-dynamic.yml.j2
│   └── smb.conf.j2
└── README.md            # This file
```

## 🎯 Quick Start

### 1. Configure Inventory
Edit `inventories/inventory.ini` to add your servers:
```ini
[all_servers]
your-server ansible_host=IP ansible_user=USER key_file=key.pem

[web]
your-server    # For nginx

[metrics]
your-server    # For node_exporter
```

### 2. Deploy Infrastructure
```bash
# From project root directory
cd /home/z0nk/Development/sc-infra

# Setup Docker
ansible-playbook -i ansible/inventories/inventory.ini ansible/playbooks/setup/simple-docker-install.yml

# Deploy services
ansible-playbook -i ansible/inventories/inventory.ini ansible/playbooks/deploy/deploy-services.yml
```

## 📂 Directory Details

### `inventories/`
- **Main config location**
- `inventory.ini` - Server definitions and service assignments
- **Edit frequently** when adding/removing servers

### `playbooks/`
- **Organized by function**
- `install/` - Individual service installations
- `deploy/` - Orchestrated deployments
- `setup/` - Infrastructure preparation
- `utils/` - Maintenance and verification

### `keys/`
- **SSH private keys** for server access
- Referenced in inventory as `key_file=filename.pem`
- **Keep secure** - never commit to git

### `configs/`
- `ansible.cfg` - Ansible behavior settings
- **Rarely changed** configuration

### `group_vars/` & `host_vars/`
- **Advanced configuration** (currently unused)
- For complex variable management

### `templates/`
- **Jinja2 templates** for dynamic configuration files
- Used by playbooks to generate configs

## 🔄 Migration Notes

**Updated paths due to reorganization:**

| Old Path | New Path |
|----------|----------|
| `ansible/inventory.ini` | `ansible/inventories/inventory.ini` |
| `ansible/playbooks/install-nginx.yml` | `ansible/playbooks/install/install-nginx.yml` |
| `ansible/playbooks/deploy-all.yml` | `ansible/playbooks/deploy/deploy-all.yml` |
| `ansible/ansible.cfg` | `ansible/configs/ansible.cfg` |

## 📖 Documentation

- **Inventory Management**: See `/docs/INVENTORY-GUIDE.md`
- **Quick Commands**: See `/docs/QUICK-REFERENCE.md`
- **Playbooks Details**: See `playbooks/README.md`

## 🛠️ Common Tasks

### Test Connections
```bash
ansible -i ansible/inventories/inventory.ini all_servers -m ping
```

### Deploy New Server
1. Add to `inventories/inventory.ini`
2. Add to appropriate service groups
3. Run deployment:
```bash
ansible-playbook -i ansible/inventories/inventory.ini ansible/playbooks/deploy/deploy-services.yml --limit new-server
```

### Verify Installation
```bash
ansible-playbook -i ansible/inventories/inventory.ini ansible/playbooks/utils/verify-docker.yml
```
