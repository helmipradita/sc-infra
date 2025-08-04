# Ansible Setup Documentation

## ✅ STATUS KONEKSI

**Status Koneksi:**
- **AWS Linux**: ✅ Connected as `ec2-user`
- **Ubuntu**: ✅ Connected as `ubuntu`

## 🔧 TEST KONEKSI

Untuk ping test sederhana yang berfungsi tanpa masalah module:

### Test koneksi simple
```bash
ansible all -i inventory.ini -m raw -a "echo 'pong'"
```

### Check user
```bash
ansible all -i inventory.ini -m raw -a "whoami"
```

### Check sistem info
```bash
ansible all -i inventory.ini -m raw -a "uname -a"
```

## 📋 PLAYBOOK YANG SUDAH DIBUAT

### 1. Check Docker status saja:
```bash
ansible-playbook -i inventory.ini playbooks/check-docker-status.yml
```

### 2. Check dan install Docker:
```bash
ansible-playbook -i inventory.ini playbooks/simple-docker-install.yml
```
