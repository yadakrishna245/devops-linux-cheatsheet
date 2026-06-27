# Day 15 — Git, Bash Scripts & Series Wrap-Up 📝

## LinkedIn Post:

```
📝 [Day 15/15] DevOps Cheat Sheet: Git, Bash & FINAL DAY!

15 days. 400+ commands. 16 topics.

Let's close this series with Git tricks & Bash scripts
that I use in production every single week:

━━━━━━━━━━━━━━━━━

# Git — beyond the basics
git log --oneline --graph --all --decorate    # Visual history
git stash                                      # Save work, switch context
git stash pop                                  # Get it back
git cherry-pick <commit-hash>                  # Grab specific commits
git reset --soft HEAD~1                        # Undo commit, keep changes
git reflog                                     # Recovery when you mess up

━━━━━━━━━━━━━━━━━

# Health Check Script (runs on all my servers):
#!/bin/bash
cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2)}')
mem=$(free | awk '/Mem/{printf("%.0f"), $3/$2*100}')
disk=$(df / | awk 'NR==2{print int($5)}')

echo "CPU: ${cpu}% | MEM: ${mem}% | DISK: ${disk}%"
[ $cpu -gt 80 ] && echo "⚠️ CPU HIGH"
[ $mem -gt 85 ] && echo "⚠️ MEM HIGH"
[ $disk -gt 90 ] && echo "⚠️ DISK HIGH"

━━━━━━━━━━━━━━━━━

# Backup with auto-cleanup:
#!/bin/bash
tar -czf "/backup/app_$(date +%Y%m%d).tar.gz" /opt/app
find /backup -name "app_*.tar.gz" -mtime +7 -delete

━━━━━━━━━━━━━━━━━

🎉 SERIES COMPLETE!

What we covered in 15 days:
🐧 Linux Admin → 👤 Users → 💾 LVM → 🌐 Network
→ ⚙️ Systemd → 📊 Performance → 🐳 Docker
→ ☸️ K8s → ☁️ AWS → 🏗️ Terraform → 📦 Ansible
→ 🔄 Jenkins → 🔐 Security → 📝 Git & Scripts

All from managing 4000+ production servers.

Plus: 200 Interview Q&A for L2/L3 roles.

🔗 Everything is in ONE repo (link in comments).

Thank you for following along! 🙏

♻️ Repost the full series to help others
#DevOps #Linux #SRE #CloudEngineering #SystemAdministration
#Docker #Kubernetes #AWS #Terraform #OpenSource
```

## First Comment:

```
🔗 FULL REPO: https://github.com/yadakrishna245/devops-linux-cheatsheet

What's inside:
→ 400+ production commands (README)
→ 200 Linux Interview Q&A (L2/L3)
→ Linux Admin Reference Guide
→ ESXi & KVM Complete Guide

⭐ If this series helped you, star the repo — it helps others find it!

What topic should I deep-dive into next? Comment below! 👇
```
