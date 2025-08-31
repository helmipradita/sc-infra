# ⚡ Quick Reference

## 🎯 Common Commands

### Deploy Services
```bash
# Deploy all services according to inventory
ansible-playbook -i ansible/inventory.ini ansible/playbooks/deploy-services.yml

# Deploy specific services
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-nginx.yml
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-grafana.yml
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-prometheus.yml
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-node-exporter-docker.yml

# Install Docker (prerequisite)
ansible-playbook -i ansible/inventory.ini ansible/playbooks/simple-docker-install.yml
```

### Test & Verify
```bash
# Test connections
ansible -i ansible/inventory.ini all_servers -m ping

# Verify Docker installation
ansible-playbook -i ansible/inventory.ini ansible/playbooks/verify-docker.yml

# Check inventory structure
ansible-inventory -i ansible/inventory.ini --list --yaml
```

### Target Specific Groups
```bash
# Deploy only to web servers
ansible-playbook -i ansible/inventory.ini playbooks/install-nginx.yml --limit web

# Deploy only to monitoring servers
ansible-playbook -i ansible/inventory.ini playbooks/install-grafana.yml --limit monitoring

# Deploy only to Ubuntu servers
ansible-playbook -i ansible/inventory.ini playbooks/simple-docker-install.yml --limit ubuntu
```

## 🏗️ Setup New Environment

### 1. Add New Ubuntu Server
```ini
# Edit ansible/inventory.ini
[all_servers]
new-ubuntu ansible_host=NEW_IP ansible_user=ubuntu key_file=key-name.pem

[web]
new-ubuntu    # For nginx

[metrics] 
new-ubuntu    # For node_exporter
```

### 2. Add New AWS Linux Server
```ini
# Edit ansible/inventory.ini
[all_servers]
new-aws ansible_host=NEW_IP ansible_user=ec2-user key_file=key-name.pem

[metrics]
new-aws       # For node_exporter
```

### 3. Deploy Everything
```bash
# 1. Install Docker first
ansible-playbook -i ansible/inventory.ini ansible/playbooks/simple-docker-install.yml --limit new-ubuntu

# 2. Install services
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-nginx.yml --limit new-ubuntu
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-node-exporter-docker.yml --limit new-ubuntu

# 3. Verify
ansible -i ansible/inventory.ini new-ubuntu -m ping
```

## 📊 Service Access

### Default URLs
- **Grafana**: http://IP:3000 (admin/admin123)
- **Prometheus**: http://IP:9090
- **Node Exporter**: http://IP:9100/metrics
- **Nginx**: http://IP (port 80)

### Service Ports
| Service | Port | Purpose |
|---------|------|---------|
| nginx | 80 | Web server |
| grafana | 3000 | Monitoring dashboard |
| prometheus | 9090 | Metrics collector |
| node_exporter | 9100 | System metrics |

## 🔧 Troubleshooting

### SSH Connection Issues
```bash
# Common SSH errors and quick fixes

# Error: "Load key: invalid format"
ssh-keygen -p -f ansible/keys/key-file.pem -m PEM -N ""

# Error: "Permission denied (publickey)"
chmod 600 ansible/keys/*.pem  # Fix permissions
ansible-inventory -i ansible/inventories/inventory.ini --host server-name  # Check config

# Test manual SSH connection
ssh -i ansible/keys/key-name.pem ubuntu@IP

# Extract public key from private key (to add to server)
ssh-keygen -y -f ansible/keys/key-name.pem
```

### Service Not Starting
```bash
# Check Docker status
ansible -i ansible/inventories/inventory.ini server-name -m raw -a "docker ps"

# Check service logs
ansible -i ansible/inventories/inventory.ini server-name -m raw -a "docker logs container-name"
```

### Variables Not Working
```bash
# Debug specific host variables
ansible-inventory -i ansible/inventories/inventory.ini --host server-name

# Test specific variable
ansible -i ansible/inventories/inventory.ini server-name -m debug -a "var=install_nginx"
```

### SSH Key Setup for New Server
```bash
# For servers created without default key pair:

# Method 1: Generate custom key (PEM format)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/custom-key -N "" -m PEM

# Method 2: Use existing key (recommended)
ssh-keygen -y -f ansible/keys/aws-cloudhelmipradita.pem
# Copy output to server's ~/.ssh/authorized_keys

# Method 3: Access via AWS Console Browser
# AWS Console → EC2 → Instance → Connect → EC2 Instance Connect
```

## 📁 File Structure
```
sc-infra/
├── ansible/
│   ├── inventory.ini           # Main configuration
│   ├── keys/                   # SSH keys
│   │   └── *.pem
│   └── playbooks/              # Ansible playbooks
│       ├── install-*.yml
│       └── deploy-*.yml
├── monitoring/                 # Config templates
│   ├── grafana/
│   ├── prometheus/
│   └── node_exporter/
└── docs/                       # Documentation
    ├── INVENTORY-GUIDE.md
    └── QUICK-REFERENCE.md
```

## 🆘 Emergency Commands

### Stop All Services
```bash
ansible -i ansible/inventory.ini all_servers -m raw -a "docker stop \$(docker ps -aq)"
```

### Restart All Services
```bash
ansible -i ansible/inventory.ini all_servers -m raw -a "docker restart \$(docker ps -aq)"
```

### Clean Docker
```bash
ansible -i ansible/inventory.ini all_servers -m raw -a "docker system prune -f"
```