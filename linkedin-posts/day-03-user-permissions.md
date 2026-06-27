# Day 3 — User & Permission Management 👤

## LinkedIn Post:

```
👤 [Day 3/15] DevOps Cheat Sheet: User & Permission Management

"Why can't I access that file?"
"Why did the app break after deployment?"

90% of the time → it's a permission issue.

Here's how I manage users & permissions on 4000+ servers:

━━━━━━━━━━━━━━━━━

# Create user with proper groups
useradd -m -s /bin/bash -G wheel username
usermod -aG docker username

# SSH key setup (stop using passwords!)
ssh-keygen -t ed25519 -C "admin@server"
ssh-copy-id user@remote-host

# SSH Tunneling — the hidden superpower:
ssh -L 8080:localhost:3306 user@remote    # Access remote DB locally
ssh -R 9090:localhost:80 user@remote      # Expose local app to remote
ssh -D 1080 user@remote                   # SOCKS proxy

# Passwordless sudo (for service accounts)
echo "deploy ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/deploy

# Fix permission issues fast
chmod 750 /opt/app
chown -R appuser:appgroup /opt/app
setfacl -m u:username:rwx /shared/dir

━━━━━━━━━━━━━━━━━

💡 Pro tip: Always use `visudo` to edit sudoers.
One typo in /etc/sudoers can lock you out of sudo entirely.

I learned this the hard way at 3AM. Don't be me. 😂

♻️ Repost to save someone from permission hell
#Linux #DevOps #SysAdmin #SSH #Security #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Star = Bookmark for interviews!

Tomorrow: LVM & Disk Management — the skill that separates junior from senior admins 💾
```
