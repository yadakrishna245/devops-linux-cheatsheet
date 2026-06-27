# Day 14 — Security Hardening 🔐

## LinkedIn Post:

```
🔐 [Day 14/15] DevOps Cheat Sheet: Security Hardening

Your server is on the internet.
Bots are trying to brute-force SSH RIGHT NOW.

Harden it in 5 minutes:

━━━━━━━━━━━━━━━━━

# 1. SSH Hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3
ClientAliveInterval 300
AllowUsers admin deploy

# 2. Fail2ban — auto-ban attackers
yum install fail2ban -y
systemctl enable --now fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
bantime = 3600
maxretry = 3

# 3. Audit critical files
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers
ausearch -k identity    # See who changed what

# 4. Disable unused services
systemctl disable --now cups bluetooth avahi-daemon

# 5. Kernel hardening
cat <<EOF >> /etc/sysctl.d/99-hardening.conf
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
kernel.randomize_va_space = 2
EOF
sysctl --system

━━━━━━━━━━━━━━━━━

💡 Pro tip: Minimum security checklist for ANY server:

✅ SSH keys only (no passwords)
✅ Fail2ban active
✅ Firewall ON (default deny)
✅ Root login disabled
✅ Unused services stopped
✅ Regular patching scheduled

Takes 5 minutes. Prevents 95% of common attacks.

♻️ Repost — security is everyone's responsibility
#Security #Linux #DevOps #CyberSecurity #Hardening #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Full security section + SELinux commands included!

Tomorrow: Final Day — Interview Q&A, Git tricks & Bash scripts! 📝
```
