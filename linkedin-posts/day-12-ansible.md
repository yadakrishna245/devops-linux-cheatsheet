# Day 12 — Ansible 📦

## LinkedIn Post:

```
📦 [Day 12/15] DevOps Cheat Sheet: Ansible

"SSH into 4000 servers and run a command on each one."

Manually? That's 4000 SSH sessions.
With Ansible? One command. 2 minutes. Done.

━━━━━━━━━━━━━━━━━

# Ad-hoc commands (quick wins)
ansible all -m ping                          # Test connectivity
ansible webservers -m shell -a "uptime"      # Check uptime
ansible all -m yum -a "name=httpd state=latest" --become
ansible all -m service -a "name=httpd state=started" --become

# Vault — encrypt your secrets
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-playbook site.yml --ask-vault-pass

━━━━━━━━━━━━━━━━━

# Playbook pattern I use everywhere:
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    app_packages:
      - nginx
      - python3
      - git

  tasks:
    - name: Install packages
      yum:
        name: "{{ app_packages }}"
        state: present

    - name: Start nginx
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

━━━━━━━━━━━━━━━━━

💡 Pro tip: Handlers only run ONCE at the end,
even if notified multiple times.

This prevents nginx from restarting 50 times
during a playbook run. Elegant design. 🎯

♻️ Repost for the automation community
#Ansible #Automation #DevOps #ConfigManagement #SRE #Linux
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Includes inventory patterns & more playbook examples!

Tomorrow: Jenkins CI/CD — complete Jenkinsfile template 🔄
```
