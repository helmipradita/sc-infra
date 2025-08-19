# 📁 Playbooks Directory Structure

## 📂 Folder Organization

### 🔧 `install/`
**Individual service installation playbooks**
- `install-nginx.yml` - Install nginx web server
- `install-grafana.yml` - Install Grafana monitoring dashboard
- `install-prometheus.yml` - Install Prometheus metrics collector
- `install-node-exporter-docker.yml` - Install Node Exporter (Docker version)
- `install-node-exporter.yml` - Install Node Exporter (Service version)
- `install-loki.yml` - Install Loki log aggregation
- `install-promtail.yml` - Install Promtail log shipper
- `install-postgres-exporter.yml` - Install PostgreSQL metrics exporter
- `install-samba.yml` - Install Samba file sharing
- `install-samba-exporter.yml` - Install Samba metrics exporter

### 🚀 `deploy/`
**Deployment orchestration playbooks**
- `deploy-all.yml` - Deploy everything according to inventory
- `deploy-services.yml` - Deploy main services (nginx, grafana, prometheus, node_exporter)
- `deploy-monitoring-stack.yml` - Deploy complete monitoring stack
- `deploy-node-exporter.yml` - Deploy node_exporter to all metrics servers

### ⚙️ `setup/`
**Infrastructure setup playbooks**
- `simple-docker-install.yml` - Install Docker and Docker Compose

### 🛠️ `utils/`
**Utility and maintenance playbooks**
- `verify-docker.yml` - Verify Docker installation
- `uninstall-node-exporter-service.yml` - Remove node_exporter service

## 🎯 Usage Examples

### Install Individual Services
```bash
# Install nginx only
ansible-playbook -i ../inventories/inventory.ini install/install-nginx.yml

# Install monitoring stack components
ansible-playbook -i ../inventories/inventory.ini install/install-grafana.yml
ansible-playbook -i ../inventories/inventory.ini install/install-prometheus.yml
```

### Deploy Complete Stacks
```bash
# Deploy everything
ansible-playbook -i ../inventories/inventory.ini deploy/deploy-all.yml

# Deploy only main services
ansible-playbook -i ../inventories/inventory.ini deploy/deploy-services.yml
```

### Setup Infrastructure
```bash
# Setup Docker first
ansible-playbook -i ../inventories/inventory.ini setup/simple-docker-install.yml
```

### Utilities
```bash
# Verify Docker installation
ansible-playbook -i ../inventories/inventory.ini utils/verify-docker.yml

# Clean up services
ansible-playbook -i ../inventories/inventory.ini utils/uninstall-node-exporter-service.yml
```

## 📋 Recommended Workflow

1. **Setup Infrastructure**
   ```bash
   ansible-playbook -i ../inventories/inventory.ini setup/simple-docker-install.yml
   ```

2. **Deploy Services**
   ```bash
   ansible-playbook -i ../inventories/inventory.ini deploy/deploy-services.yml
   ```

3. **Verify Installation**
   ```bash
   ansible-playbook -i ../inventories/inventory.ini utils/verify-docker.yml
   ```

## 🔄 Path Updates

**Note**: Due to restructuring, update your commands to use new paths:

**Before:**
```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/install-nginx.yml
```

**After:**
```bash
ansible-playbook -i ansible/inventories/inventory.ini ansible/playbooks/install/install-nginx.yml
```