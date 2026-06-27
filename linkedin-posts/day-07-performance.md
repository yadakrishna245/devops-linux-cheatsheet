# Day 7 — Performance & Troubleshooting 📊

## LinkedIn Post:

```
📊 [Day 7/15] DevOps Cheat Sheet: Performance & Troubleshooting

Server is slow. Everyone is panicking.

Don't guess. Run these 5 commands in order:

━━━━━━━━━━━━━━━━━

# 1. Quick overview — is it CPU, Memory, or I/O?
uptime                                  # Load average
top -bn1 | head -20                     # CPU & memory snapshot

# 2. Who's eating CPU?
ps aux --sort=-%cpu | head -20

# 3. Who's eating Memory?
ps aux --sort=-%mem | head -20

# 4. Is the disk choking?
iostat -xz 1 5
iotop -o                                # Real-time disk I/O per process

# 5. OOM killer struck?
dmesg | grep -i "out of memory"
grep -i "killed process" /var/log/messages

━━━━━━━━━━━━━━━━━

# Bonus — find what's using a specific port
lsof -i :8080

# Trace a process's system calls
strace -p <PID> -e trace=network

# Detailed CPU/Memory over time
sar -u 1 5                              # CPU
sar -r 1 5                              # Memory
vmstat 1 5                              # Combined view

━━━━━━━━━━━━━━━━━

💡 Pro tip: Load average context matters!

Load avg of 8 on a 2-CPU server = CRITICAL 🔥
Load avg of 8 on a 16-CPU server = perfectly fine ✅

Always check: nproc (number of CPUs)

Rule: Load avg > nproc = you have a problem.

♻️ Repost — this sequence saves hours of debugging
#Linux #Performance #DevOps #Troubleshooting #SRE #SysAdmin
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Star it — these commands will save you during incidents!

Tomorrow: Docker — multi-stage builds & container management 🐳
```
