# Day 6 — Systemd & Services ⚙️

## LinkedIn Post:

```
⚙️ [Day 6/15] DevOps Cheat Sheet: Systemd & Services

"The app crashed and nobody restarted it."

If you're still manually restarting services after crashes,
you're doing it wrong.

Here's how systemd does it automatically:

━━━━━━━━━━━━━━━━━

# Essential service management
systemctl start|stop|restart|status <service>
systemctl enable --now <service>
systemctl list-units --type=service --state=running
systemctl list-timers --all

# Check logs like a pro
journalctl -u <service> -f                  # Follow live
journalctl -u <service> --since "1 hour ago"
journalctl -p err -b                        # Errors since boot

# CREATE YOUR OWN SERVICE (copy-paste ready):
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

━━━━━━━━━━━━━━━━━

💡 Pro tip: `Restart=always` + `RestartSec=5`

This means your app auto-restarts within 5 seconds of crashing.
No pager alerts at 3AM. No manual intervention.

Let systemd be your first responder. 🚒

♻️ Repost — every DevOps engineer needs custom services
#Linux #Systemd #DevOps #SRE #Automation #SysAdmin
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ 400+ commands — star it for your reference!

Tomorrow: Performance & Troubleshooting — find the bottleneck in 60 seconds 📊
```
