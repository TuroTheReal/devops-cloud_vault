# Ansible - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, ansible, automation]
created: 2025-01-21
version: 2.9+
```

**Official docs**: [docs.ansible.com](https://docs.ansible.com/)
**Related concepts**: [[concepts/ansible/ansible-fundamentals]]

---

## 📑 Table of Contents

1. [Quick Start](#-quick-start)
2. [Commands](#-commands)
3. [Playbook Execution](#-playbook-execution)
4. [Debug / Inspection](#-debug--inspection)
5. [Ansible Vault](#-ansible-vault)
6. [Useful One-Liners](#-useful-one-liners)
7. [Modules Reference](#-modules-reference)
8. [Config Example](#-config-example)

---

## 🚀 Quick Start

```bash
# Installation
pip install ansible
# or
sudo apt install ansible

# Install Docker collection (community)
ansible-galaxy collection install community.docker

# Test connection
ansible all -i inventory.yml -m ping

# Run a playbook
ansible-playbook -i inventory.yml playbook.yml
```

---

## 💻 Commands

### Ad-hoc Commands
```bash
# Ping all hosts
ansible all -i inventory.yml -m ping

# Execute a command
ansible all -i inventory.yml -m command -a "uptime"

# With shell (pipes, redirections)
ansible all -i inventory.yml -m shell -a "df -h | head -5"

# Install a package
ansible all -i inventory.yml -m apt -a "name=nginx state=present" --become

# Copy a file
ansible all -i inventory.yml -m copy -a "src=file.txt dest=/tmp/file.txt"

# Target a group
ansible webservers -i inventory.yml -m ping
```

---

## 🎬 Playbook Execution

```bash
# Run a playbook
ansible-playbook -i inventory.yml playbook.yml

# Dry-run (simulate without applying)
ansible-playbook playbook.yml --check

# Show differences
ansible-playbook playbook.yml --diff

# Check + diff combined
ansible-playbook playbook.yml --check --diff

# Run only certain tags
ansible-playbook playbook.yml --tags "nginx,docker"

# Exclude tags
ansible-playbook playbook.yml --skip-tags "slow"

# Pass a variable via CLI
ansible-playbook playbook.yml -e "http_port=8080"

# Limit to certain hosts
ansible-playbook playbook.yml --limit "web1,web2"

# Verbose (debug)
ansible-playbook playbook.yml -v    # Basic
ansible-playbook playbook.yml -vv   # More
ansible-playbook playbook.yml -vvv  # Maximum
```

---

## 🔍 Debug / Inspection

```bash
# List inventory hosts
ansible-inventory -i inventory.yml --list

# Inventory graph
ansible-inventory -i inventory.yml --graph

# List playbook tasks
ansible-playbook playbook.yml --list-tasks

# List tags
ansible-playbook playbook.yml --list-tags

# List targeted hosts
ansible-playbook playbook.yml --list-hosts

# Syntax check
ansible-playbook playbook.yml --syntax-check
```

---

## 🔐 Ansible Vault

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View content (without editing)
ansible-vault view secrets.yml

# Decrypt
ansible-vault decrypt secrets.yml

# Change password
ansible-vault rekey secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
# or with password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

---

## ⚡ Useful One-Liners

```bash
# Reboot all servers
ansible all -i inventory.yml -m reboot --become

# Get host facts
ansible web1 -i inventory.yml -m setup

# Filtered facts (e.g.: IP)
ansible web1 -m setup -a "filter=ansible_default_ipv4"

# Run playbook step-by-step
ansible-playbook playbook.yml --step

# Start at specific task
ansible-playbook playbook.yml --start-at-task="Install nginx"
```

---

## 📦 Modules Reference

### Package Management
```yaml
# APT (Debian/Ubuntu)
- apt:
    name: nginx
    state: present      # present, absent, latest
    update_cache: yes

# Multiple packages
- apt:
    name:
      - nginx
      - git
      - curl
    state: present
```

### Service Management
```yaml
# Systemd
- systemd:
    name: nginx
    state: started      # started, stopped, restarted, reloaded
    enabled: yes        # Start on boot

# Service (generic)
- service:
    name: nginx
    state: started
```

### Files
```yaml
# Copy file
- copy:
    src: local_file.txt
    dest: /remote/path/file.txt
    owner: root
    mode: '0644'

# Jinja2 template
- template:
    src: config.j2
    dest: /etc/app/config.conf

# Create directory
- file:
    path: /opt/myapp
    state: directory
    mode: '0755'

# Download file
- get_url:
    url: https://example.com/file.tar.gz
    dest: /tmp/file.tar.gz
```

### Docker
```yaml
# Docker Compose v2 (community.docker)
- community.docker.docker_compose_v2:
    project_src: /opt/app
    state: present

# Exec in container
- community.docker.docker_container_exec:
    container: myapp
    command: /bin/bash -c "echo hello"
```

### Users
```yaml
# Create user
- user:
    name: deploy
    groups: docker,sudo
    append: yes
    shell: /bin/bash

# Add SSH key
- authorized_key:
    user: deploy
    key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

---

## 📝 Config Example

### inventory.yml
```yaml
all:
  hosts:
    web1:
      ansible_host: 1.2.3.4
    web2:
      ansible_host: 5.6.7.8
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/aws
    ansible_python_interpreter: /usr/bin/python3
```

### playbook.yml
```yaml
---
- name: Setup web servers
  hosts: all
  become: yes

  vars:
    app_port: 80

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install packages
      apt:
        name:
          - nginx
          - docker.io
        state: present

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    - name: Ensure nginx running
      systemd:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart nginx
      systemd:
        name: nginx
        state: restarted
```

### group_vars/all.yml
```yaml
---
# Global variables
app_name: myapp
app_port: 80
deploy_user: ubuntu

# Docker
docker_compose_version: "2.24.6"
```

---

**Last update**: 2025-01-21
**Tool version**: 2.9+
