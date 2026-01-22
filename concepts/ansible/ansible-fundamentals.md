# Ansible Fundamentals - Configuration Management

## 📋 Metadata

```yaml
tags: [concept, ansible, automation, status/learned]
created: 2025-01-21
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 12h
```

**Prerequisites**: [[concepts/linux/linux-basics]], [[concepts/linux/linux-ssh-hardening]]
**Related to**: [[concepts/terraform/terraform-fundamentals]]
**Official docs**: [Ansible Docs](https://docs.ansible.com/) | [Module Index](https://docs.ansible.com/ansible/latest/collections/index.html) | [Roles Guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)

---

## 🎯 TL;DR (30 seconds)

Ansible configures servers in an automated and idempotent way. You describe the desired state, Ansible ensures the server reaches (or stays in) that state.

**Analogy**: If Terraform builds the house, Ansible decorates and furnishes it.

---

## 🤔 When to Use?

### ✅ Good for
1. **Server configuration**: Install packages, configure services, deploy apps
2. **Multi-server**: Same config on 1 or 100 servers
3. **Idempotence**: Run 10 times = same result

### ❌ Bad for
- **Create infrastructure** → Use [[concepts/terraform/terraform-fundamentals]] instead
- **Containers** → Docker/Kubernetes more suitable

---

## 📚 Key Concepts

### 1. Playbook

**My understanding**:
A playbook describes WHAT to do and in what ORDER. It's a list of organized tasks that execute sequentially on targeted hosts.

**Why important**:
It's the heart of Ansible. No playbook, no automation.

**Mental model**:
```yaml
- name: Playbook description
  hosts: which servers
  become: with sudo or not
  tasks:
    - name: Task 1
      module: parameters
    - name: Task 2
      module: parameters
```

---

### 2. Inventory

**My understanding**:
The inventory describes WHO (the hosts) and HOW (the connections). It's the list of servers with their connection variables.

**Why important**:
Without inventory, Ansible doesn't know where to connect.

**Mental model**:
```yaml
# inventory.yml
all:
  hosts:
    web1:
      ansible_host: 1.2.3.4
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ~/.ssh/key
    web2:
      ansible_host: 5.6.7.8
```

---

### 3. Idempotence

**My understanding**:
Running the playbook N times always gives the same result. If the server is already in the desired state, Ansible does nothing (changed=0).

**Why important**:
You can re-run without fear of breaking things. No "I already installed nginx, it's going to crash".

**Mental model**:
```
1st execution: changed=5 (5 modifications)
2nd execution: changed=0 (already OK)
3rd execution: changed=0 (still OK)
```

---

### 4. Modules

**My understanding**:
Modules are Ansible's "verbs". Each module performs a specific action (install package, copy file, manage service...).

**Why important**:
The right modules = guaranteed idempotence. Avoid raw shell commands.

**Docs**: [All Collections/Modules](https://docs.ansible.com/ansible/latest/collections/index.html)

**Common modules**:
| Module | Action |
|--------|--------|
| `apt` | Install/remove packages |
| `systemd` | Manage services (start/stop/enable) |
| `copy` | Copy files |
| `template` | Copy with Jinja2 variables |
| `file` | Create folders, permissions |
| `docker_compose_v2` | Manage Docker Compose |

---

### 5. Roles

**My understanding**:
Roles are reusable, shareable Ansible code packages. Like Terraform modules - don't reinvent the wheel.

**Why important**:
Structure your playbooks, share code across projects, use community roles.

**Docs**: [Roles Guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) | [Ansible Galaxy (community roles)](https://galaxy.ansible.com/)

**Structure**:
```
roles/
└── nginx/
    ├── tasks/main.yml      # What to do
    ├── handlers/main.yml   # Handlers (restart etc.)
    ├── templates/          # Jinja2 templates
    ├── files/              # Static files
    ├── vars/main.yml       # Variables
    └── defaults/main.yml   # Default values
```

---

### 6. Handlers

**My understanding**:
Handlers are tasks that execute ONLY if notified by another task that changed something. Perfect for "restart nginx if config modified".

**Why important**:
Avoids unnecessary restarts. Service should restart only when needed.

**Mental model**:
```yaml
tasks:
  - name: Copy nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx  # Triggers if changed

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

---

## 💻 Minimal Example

### Context
Install Docker and deploy a container on a server.

### Code
```yaml
# playbook.yml
---
- name: Install Docker and deploy app
  hosts: all
  become: yes  # Like sudo

  tasks:
    - name: Check if Docker installed
      command: docker --version
      register: docker_check
      ignore_errors: yes
      changed_when: false

    - name: Install Docker
      apt:
        name: docker.io
        state: present
        update_cache: yes
      when: docker_check.rc != 0

    - name: Ensure Docker running
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Add user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Show Docker version
      command: docker --version
      register: docker_version
      changed_when: false

    - name: Result
      debug:
        msg: "Docker ready: {{ docker_version.stdout }}"
```

**Output**: Docker installed and started, user added to docker group.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Different Docker versions between environments

**Symptom**: Playbook works on test VPS but fails on EC2 (different docker modules)

**What I did wrong**:
```yaml
# ❌ Wrong - Use docker_container without checking the version
- name: Run container
  docker_container:
    name: app
    image: nginx
```

**Solution**:
```yaml
# ✅ Correct - Check version first and use the right module
- name: Check Docker version first
  command: docker --version
  register: docker_ver

# Or use community.docker.docker_compose_v2
# which is more stable cross-versions
- name: Deploy with compose
  community.docker.docker_compose_v2:
    project_src: /opt/app
    state: present
```

**Time wasted**: 1h30
**Lesson**: Always test on an environment identical to production. Personal VPS != fresh EC2 Ubuntu.

---

## 📊 Stats

```yaml
Total time: 12h (55% assisted / 45% autonomous)
Used in: [[projects/2025-01-aws-terraform-ansible/learnings]]
```

---

**Last update**: 2025-01-21
**Next review**: 2025-02-21
