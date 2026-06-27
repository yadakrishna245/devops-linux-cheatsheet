# Day 5 — Network & Firewall 🌐

## LinkedIn Post:

```
🌐 [Day 5/15] DevOps Cheat Sheet: Network & Firewall

"The app works on my machine!"
"The server can't reach the database!"

Networking issues cause 40% of production incidents.

Here's my troubleshooting toolkit:

━━━━━━━━━━━━━━━━━

# Diagnostics — run these FIRST
ip addr show                         # What IPs do I have?
ip route show                        # Where does traffic go?
ss -tulnp                            # What ports are listening?
nethogs eth0                         # Bandwidth per process (real-time)

# DNS debugging
dig google.com
dig +trace google.com                # Full resolution path
cat /etc/resolv.conf

# Firewalld (RHEL/CentOS — use this)
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

# iptables (legacy but still everywhere)
iptables -L -n -v
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP

# Network bonding (HA for NICs)
nmcli con add type bond con-name bond0 ifname bond0 mode active-backup

━━━━━━━━━━━━━━━━━

💡 Pro tip: Use `ss -tulnp` not `netstat`.
ss is faster, more accurate, and netstat is deprecated.

Also: always check `firewall-cmd --list-all` BEFORE
blaming the application. 9/10 times it's a blocked port. 🤦

♻️ Repost for your DevOps network
#Linux #Networking #Firewall #DevOps #SysAdmin #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Star it — your future self will thank you!

Tomorrow: Systemd & Services — create custom services like a pro ⚙️
```
