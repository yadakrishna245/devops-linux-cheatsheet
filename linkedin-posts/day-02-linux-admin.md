# Day 2 — Linux Administration 🐧

## LinkedIn Post:

```
🐧 [Day 2/15] DevOps Cheat Sheet: Linux Administration

You SSH into a new server. What's the FIRST thing you check?

Here's my exact workflow:

━━━━━━━━━━━━━━━━━

# 1. What OS am I on?
cat /etc/os-release

# 2. How long has it been up? Any load issues?
uptime

# 3. CPU & Memory at a glance
lscpu
free -h

# 4. Disk space — the silent killer
df -hT
lsblk

# 5. Find and kill old logs eating disk
find / -name "*.log" -mtime +7 -delete

# 6. What's eating the most space?
du -h --max-depth=1 / | sort -hr | head -20

━━━━━━━━━━━━━━━━━

💡 Pro tip: Set up dnf-automatic for security patches:

dnf install dnf-automatic -y
systemctl enable --now dnf-automatic-install.timer

Then check if reboot is needed:
needs-restarting -r

I've seen servers running 500+ days without patches.
Don't be that team. 😅

🔗 Full cheatsheet in comments 👇

♻️ Repost if your network needs this
#Linux #DevOps #SysAdmin #CloudEngineering #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Star it to bookmark for later!

Tomorrow: User & Permission Management — SSH tunneling tricks included 🔐
```
