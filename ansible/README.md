✅ PING BERHASIL!

Status Koneksi:
- AWS Linux: ✅ Connected as ec2-user
- Ubuntu: ✅ Connected as ubuntu

Untuk ping test sederhana yang berfungsi tanpa masalah module:

# Test koneksi simple
ansible all -i ansible/inventory.ini -m raw -a "echo 'pong'"

# Check user
ansible all -i ansible/inventory.ini -m raw -a "whoami"

# Check sistem info
ansible all -i ansible/inventory.ini -m raw -a "uname -a"