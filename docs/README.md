# 📚 SC-Infra Documentation

Welcome to the SC-Infra documentation! This directory contains guides for managing your infrastructure setup with Ansible.

## 📖 Documentation Files

### 1. [INVENTORY-GUIDE.md](./INVENTORY-GUIDE.md)
**Complete guide for inventory management**
- How to add new servers (Ubuntu/AWS Linux)
- Service group configurations
- Detailed examples and troubleshooting
- **Read this for understanding the system**

### 2. [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)
**Commands and quick setup guide**
- Common Ansible commands
- Service deployment commands
- Emergency troubleshooting commands
- **Use this for daily operations**

## 🚀 Getting Started

### First Time Setup
1. **Read**: [INVENTORY-GUIDE.md](./INVENTORY-GUIDE.md) to understand the system
2. **Edit**: `ansible/inventory.ini` to add your servers
3. **Deploy**: Use commands from [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)

### Daily Operations
- **Adding servers**: See examples in INVENTORY-GUIDE.md
- **Deploying services**: Use commands in QUICK-REFERENCE.md
- **Troubleshooting**: Check both guides for solutions

## 🎯 Service Overview

### Available Services
- **nginx**: Web server and reverse proxy
- **grafana**: Monitoring dashboard (port 3000)
- **prometheus**: Metrics collection (port 9090)
- **node_exporter**: System metrics (port 9100)

### Service Groups in Inventory
- `[web]`: Servers with nginx
- `[monitoring]`: Servers with grafana + prometheus
- `[metrics]`: Servers with node_exporter

## 📁 Project Structure
```
sc-infra/
├── ansible/
│   ├── inventory.ini           # 📝 Main config (edit frequently)
│   ├── keys/                   # SSH keys
│   └── playbooks/              # Deployment scripts
├── monitoring/                 # Service configurations
├── docs/                       # 📚 This documentation
└── README.md                   # Project overview
```

## 🆘 Need Help?

1. **Can't connect to server?** → Check INVENTORY-GUIDE.md troubleshooting section
2. **Service not installing?** → Check QUICK-REFERENCE.md troubleshooting section
3. **Adding new server type?** → See examples in INVENTORY-GUIDE.md
4. **Forgot a command?** → Check QUICK-REFERENCE.md for common commands

## 🔄 Quick Commands

```bash
# Test all connections
ansible -i ansible/inventory.ini all_servers -m ping

# Deploy everything
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy-services.yml

# Check what's configured
ansible-inventory -i ansible/inventory.ini --list --yaml
```

---

**💡 Tip**: Keep this documentation updated when you make changes to the infrastructure setup!