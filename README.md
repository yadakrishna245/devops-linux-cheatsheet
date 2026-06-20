<div align="center">

# 🚀 DevOps & Linux Command Cheat Sheet

[![Stars](https://img.shields.io/github/stars/yadakrishna245/devops-linux-cheatsheet?style=for-the-badge&color=yellow)](https://github.com/yadakrishna245/devops-linux-cheatsheet/stargazers)
[![Forks](https://img.shields.io/github/forks/yadakrishna245/devops-linux-cheatsheet?style=for-the-badge&color=blue)](https://github.com/yadakrishna245/devops-linux-cheatsheet/network/members)
[![License](https://img.shields.io/github/license/yadakrishna245/devops-linux-cheatsheet?style=for-the-badge)](LICENSE)

**The ultimate reference for DevOps Engineers & Linux Administrators**

*Real-world commands from managing 4000+ production servers*

[Linux](#-linux-administration) • [Docker](#-docker) • [Kubernetes](#-kubernetes) • [AWS CLI](#-aws-cli) • [Terraform](#-terraform) • [Ansible](#-ansible) • [Jenkins](#-jenkins) • [Git](#-git) • [Networking](#-networking) • [Monitoring](#-monitoring)

---

⭐ **If this helped you, give it a star!** ⭐

### 📚 Additional Resources

| Resource | Description |
|----------|-------------|
| [📝 200 Linux Interview Q&A (L2/L3)](LINUX-INTERVIEW-QA.md) | Real interview questions with detailed answers |
| [📖 Linux Admin Reference Guide](LINUX-ADMIN-REFERENCE.md) | Complete admin reference — Users, LVM, Network, Services, Security |
| [🖥️ ESXi & KVM Complete Guide](ESXI-KVM-GUIDE.md) | Beginner to Advanced — Installation on blade servers, administration, interview Q&A |

</div>

---

## 📋 Table of Contents

- [Linux Administration](#-linux-administration)
- [User & Permission Management](#-user--permission-management)
- [Disk & Storage](#-disk--storage)
- [Network & Firewall](#-network--firewall)
- [Systemd & Services](#-systemd--services)
- [Performance & Troubleshooting](#-performance--troubleshooting)
- [Docker](#-docker)
- [Kubernetes](#-kubernetes)
- [AWS CLI](#-aws-cli)
- [Terraform](#-terraform)
- [Ansible](#-ansible)
- [Jenkins](#-jenkins)
- [Git](#-git)
- [Monitoring & Logs](#-monitoring--logs)
- [Security Hardening](#-security-hardening)
- [Bash Scripting Snippets](#-bash-scripting-snippets)

---

## 🐧 Linux Administration

### System Information

```bash
# OS and kernel info
uname -a
cat /etc/os-release
hostnamectl

# Hardware info
lscpu
free -h
df -hT
lsblk

# Uptime and load
uptime
cat /proc/loadavg
```

### Package Management

```bash
# RHEL/CentOS (yum/dnf)
yum update -y
yum install <package> -y
yum remove <package>
yum list installed | grep <package>
dnf module list

# Ubuntu/Debian (apt)
apt update && apt upgrade -y
apt install <package> -y
apt remove <package>
apt autoremove
dpkg -l | grep <package>
```

### File & Directory Operations

```bash
# Find files
find / -name "*.log" -mtime +7 -delete        # Delete logs older than 7 days
find / -type f -size +100M                     # Find files larger than 100MB
find / -perm -4000                             # Find SUID files

# Archive & compress
tar -czvf backup.tar.gz /path/to/dir
tar -xzvf backup.tar.gz -C /destination
rsync -avz --progress source/ dest/

# Disk usage
du -sh /var/log/*
du -h --max-depth=1 / | sort -hr | head -20
ncdu /                                         # Interactive disk usage
```

---

## 👤 User & Permission Management

```bash
# User management
useradd -m -s /bin/bash -G wheel username
usermod -aG docker username
passwd username
userdel -r username

# SSH key setup
ssh-keygen -t ed25519 -C "admin@server"
ssh-copy-id user@remote-host

# Sudoers (visudo)
echo "username ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/username

# File permissions
chmod 750 /opt/app
chown -R appuser:appgroup /opt/app
setfacl -m u:username:rwx /shared/dir

# SELinux
getenforce
setenforce 0                                   # Permissive (temp)
sestatus
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
```

---

## 💾 Disk & Storage

### LVM Management

```bash
# Physical volumes
pvcreate /dev/sdb
pvdisplay

# Volume groups
vgcreate vg_data /dev/sdb
vgextend vg_data /dev/sdc
vgdisplay

# Logical volumes
lvcreate -L 50G -n lv_app vg_data
lvextend -L +20G /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app                  # ext4
xfs_growfs /dev/vg_data/lv_app                 # xfs

# Mount permanently
echo "/dev/vg_data/lv_app /opt/app xfs defaults 0 0" >> /etc/fstab
mount -a
```

### NFS

```bash
# Server side
yum install nfs-utils -y
echo "/shared 192.168.1.0/24(rw,sync,no_root_squash)" >> /etc/exports
exportfs -rav
systemctl enable --now nfs-server

# Client side
mount -t nfs server:/shared /mnt/nfs
echo "server:/shared /mnt/nfs nfs defaults 0 0" >> /etc/fstab
```

---

## 🌐 Network & Firewall

```bash
# Network diagnostics
ip addr show
ip route show
ss -tulnp                                      # Listening ports
nmcli con show
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con up eth0

# DNS
dig google.com
nslookup google.com
cat /etc/resolv.conf

# Firewalld (RHEL/CentOS)
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

# iptables
iptables -L -n -v
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP
iptables-save > /etc/iptables/rules.v4

# Network bonding
nmcli con add type bond con-name bond0 ifname bond0 mode active-backup
nmcli con add type ethernet con-name bond0-slave1 ifname eth0 master bond0
nmcli con add type ethernet con-name bond0-slave2 ifname eth1 master bond0
```

---

## ⚙️ Systemd & Services

```bash
# Service management
systemctl start|stop|restart|status <service>
systemctl enable --now <service>
systemctl disable <service>
systemctl list-units --type=service --state=running
systemctl list-unit-files | grep enabled

# Journald logs
journalctl -u <service> -f                     # Follow logs
journalctl -u <service> --since "1 hour ago"
journalctl -p err -b                           # Errors since boot

# Create custom service
cat <<EOF > /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
ExecStart=/opt/app/start.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now myapp
```

---

## 📊 Performance & Troubleshooting

```bash
# CPU & Memory
top -bn1 | head -20
htop
vmstat 1 5
sar -u 1 5                                    # CPU usage
sar -r 1 5                                    # Memory usage

# Disk I/O
iostat -xz 1 5
iotop -o

# Process management
ps aux --sort=-%mem | head -20                 # Top memory consumers
ps aux --sort=-%cpu | head -20                 # Top CPU consumers
kill -9 <PID>
pkill -f <process_name>

# Strace & debugging
strace -p <PID> -e trace=network
lsof -i :8080                                 # What's using port 8080
lsof -u username                              # Files opened by user

# OOM killer check
dmesg | grep -i "out of memory"
grep -i "killed process" /var/log/messages
```

---


## 🐳 Docker

### Container Lifecycle

```bash
# Quick container health check
docker inspect --format='{{.State.Health.Status}}' <container>

# Run containers
docker run -d --name myapp -p 8080:80 --restart=always nginx:latest
docker run -d --name db -e MYSQL_ROOT_PASSWORD=secret -v dbdata:/var/lib/mysql mysql:8
docker run --rm -it ubuntu:22.04 /bin/bash

# Container management
docker ps -a
docker logs -f --tail 100 <container>
docker exec -it <container> /bin/bash
docker stop $(docker ps -q)                    # Stop all containers
docker rm $(docker ps -aq)                     # Remove all containers

# Images
docker build -t myapp:v1 .
docker tag myapp:v1 registry.example.com/myapp:v1
docker push registry.example.com/myapp:v1
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
docker image prune -a                          # Remove unused images

# Volumes & Networks
docker volume create app_data
docker volume ls
docker network create --driver bridge app_net
docker network inspect app_net
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - api
    restart: always

  api:
    build: ./api
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    restart: always

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: always

volumes:
  pgdata:
```

```bash
docker compose up -d
docker compose down -v
docker compose logs -f api
docker compose ps
docker compose exec api sh
```

### Dockerfile Best Practices

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

---

## ☸️ Kubernetes

### Cluster Info & Contexts

```bash
kubectl cluster-info
kubectl get nodes -o wide
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl api-resources                          # List all resource types
```

### Pods & Deployments

```bash
# Pods
kubectl get pods -A                            # All namespaces
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f --tail=100
kubectl logs <pod-name> -c <container>         # Multi-container pod
kubectl exec -it <pod-name> -- /bin/sh
kubectl delete pod <pod-name> --grace-period=0 --force

# Deployments
kubectl create deployment nginx --image=nginx:latest --replicas=3
kubectl set image deployment/nginx nginx=nginx:1.25
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
kubectl scale deployment/nginx --replicas=5

# Services
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl get svc
kubectl get endpoints
```

### Troubleshooting

```bash
# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl top nodes
kubectl describe node <node-name> | grep -A5 "Conditions"

# Debug with ephemeral container
kubectl debug -it <pod-name> --image=busybox --target=<container>

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local-file

# Resource usage
kubectl get pods --field-selector=status.phase=Failed
kubectl get pods | grep -v Running

# Quick YAML generation
kubectl run test --image=busybox --dry-run=client -o yaml > pod.yaml
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deploy.yaml
```

---


## ☁️ AWS CLI

### EC2

```bash
# List instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PrivateIpAddress,Tags[?Key==`Name`].Value|[0]]' --output table

# Start/Stop
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Security Groups
aws ec2 describe-security-groups --group-ids sg-xxx
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 443 --cidr 0.0.0.0/0
```

### S3

```bash
aws s3 ls
aws s3 cp file.txt s3://my-bucket/
aws s3 sync ./local-dir s3://my-bucket/prefix/
aws s3 rm s3://my-bucket/prefix/ --recursive
aws s3 presign s3://my-bucket/file.txt --expires-in 3600
```

### IAM & STS

```bash
aws sts get-caller-identity
aws iam list-users --output table
aws iam create-user --user-name deploy-bot
aws iam attach-user-policy --user-name deploy-bot --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

### CloudWatch & Logs

```bash
aws logs describe-log-groups
aws logs tail /aws/lambda/my-function --follow
aws logs filter-log-events --log-group-name /aws/ecs/app --filter-pattern "ERROR"
```

### RDS

```bash
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceStatus,Engine,DBInstanceClass]' --output table
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-snap-$(date +%Y%m%d)
```

---

## 🏗️ Terraform

### Essential Commands

```bash
terraform init
terraform plan -out=tfplan
terraform apply tfplan
terraform destroy
terraform state list
terraform state show aws_instance.web
terraform state mv aws_instance.old aws_instance.new
terraform state rm aws_instance.deprecated
terraform import aws_instance.web i-1234567890abcdef0
terraform fmt -recursive
terraform validate
```

### Common Patterns

```hcl
# EC2 Instance
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  subnet_id     = aws_subnet.private.id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  root_block_device {
    volume_size = 50
    volume_type = "gp3"
  }

  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}

# S3 Bucket with versioning
resource "aws_s3_bucket" "logs" {
  bucket = "company-logs-${var.environment}"
  
  tags = {
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "logs" {
  bucket = aws_s3_bucket.logs.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

---

## 📦 Ansible

### Ad-hoc Commands

```bash
# Ansible Vault (encrypt secrets)
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-vault encrypt vars.yml
ansible-vault decrypt vars.yml
ansible-playbook site.yml --ask-vault-pass

ansible all -m ping
ansible webservers -m shell -a "uptime"
ansible all -m yum -a "name=httpd state=latest" --become
ansible all -m service -a "name=httpd state=started enabled=yes" --become
ansible all -m copy -a "src=./app.conf dest=/etc/app.conf" --become
```

### Playbook Example

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    app_packages:
      - nginx
      - python3
      - git

  tasks:
    - name: Install packages
      yum:
        name: "{{ app_packages }}"
        state: present

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Inventory

```ini
# inventory.ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---


## 🔄 Jenkins

### Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'myapp'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/org/repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker run --rm ${APP_NAME}:${BUILD_NUMBER} npm test'
            }
        }
        
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login ${DOCKER_REGISTRY} -u $USER --password-stdin'
                    sh 'docker push ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}'
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#alerts', message: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        success {
            slackSend channel: '#deploys', message: "Deployed: ${APP_NAME}:${BUILD_NUMBER}"
        }
    }
}
```

---

## 🔀 Git

```bash
# Branch management
git checkout -b feature/new-feature
git branch -d feature/merged-branch
git branch -a                                  # List all branches

# Stash
git stash
git stash list
git stash pop
git stash drop stash@{0}

# Rebase & squash
git rebase main
git rebase -i HEAD~3                           # Squash last 3 commits

# Cherry-pick
git cherry-pick <commit-hash>

# Undo mistakes
git reset --soft HEAD~1                        # Undo commit, keep changes staged
git checkout -- <file>                         # Discard file changes
git reflog                                     # Recovery history

# Tags
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# Clean
git clean -fd                                  # Remove untracked files/dirs
git gc --prune=now                             # Garbage collection
```

---

## 📈 Monitoring & Logs

### Prometheus Queries (PromQL)

```promql
# CPU usage per pod
rate(container_cpu_usage_seconds_total[5m])

# Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100

# Disk space
(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes * 100

# HTTP request rate
rate(http_requests_total{status=~"5.."}[5m])

# Pod restart count
kube_pod_container_status_restarts_total > 3
```

### Log Management

```bash
# Journalctl
journalctl -u nginx --since "2024-01-01" --until "2024-01-02"
journalctl --disk-usage
journalctl --vacuum-size=500M

# Syslog
tail -f /var/log/messages
grep -i error /var/log/syslog | tail -50

# Log rotation
cat <<EOF > /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser appgroup
}
EOF
```

---

## 🔐 Security Hardening

```bash
# SSH hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers admin deploy

# Fail2ban
yum install fail2ban -y
systemctl enable --now fail2ban
# /etc/fail2ban/jail.local
[sshd]
enabled = true
bantime = 3600
maxretry = 3

# Audit important files
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers
ausearch -k identity

# Disable unused services
systemctl disable --now cups bluetooth avahi-daemon

# Kernel hardening (sysctl)
cat <<EOF >> /etc/sysctl.d/99-hardening.conf
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
kernel.randomize_va_space = 2
EOF
sysctl --system
```

---

## 📜 Bash Scripting Snippets

### Health Check Script

```bash
#!/bin/bash
# Server health check

THRESHOLD_CPU=80
THRESHOLD_MEM=85
THRESHOLD_DISK=90

cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2)}')
mem_usage=$(free | awk '/Mem/{printf("%.0f"), $3/$2*100}')
disk_usage=$(df / | awk 'NR==2{print int($5)}')

echo "=== Server Health: $(hostname) ==="
echo "CPU:  ${cpu_usage}%  $([ $cpu_usage -gt $THRESHOLD_CPU ] && echo '⚠️ HIGH')"
echo "MEM:  ${mem_usage}%  $([ $mem_usage -gt $THRESHOLD_MEM ] && echo '⚠️ HIGH')"
echo "DISK: ${disk_usage}% $([ $disk_usage -gt $THRESHOLD_DISK ] && echo '⚠️ HIGH')"
```

### Backup Script

```bash
#!/bin/bash
# Automated backup with retention

BACKUP_DIR="/backup"
SOURCE="/opt/app"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)

tar -czf "${BACKUP_DIR}/app_${DATE}.tar.gz" "$SOURCE"

# Cleanup old backups
find "$BACKUP_DIR" -name "app_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "[$(date)] Backup completed: app_${DATE}.tar.gz"
```

### Log Monitor with Alert

```bash
#!/bin/bash
# Monitor logs for errors and alert

LOG_FILE="/var/log/app/error.log"
ALERT_EMAIL="admin@example.com"

tail -Fn0 "$LOG_FILE" | while read -r line; do
    if echo "$line" | grep -qi "critical\|fatal\|out of memory"; then
        echo "$line" | mail -s "🚨 CRITICAL: $(hostname)" "$ALERT_EMAIL"
    fi
done
```

---

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file.

---

<div align="center">

**Made with ❤️ by [Krishna Chaithanya](https://github.com/yadakrishna245)**

*Senior System Administrator | 8+ Years | 4000+ Servers*

If this repo helped you, please ⭐ **star it** — it helps others find it too!

</div>
