# Ansible Inventory & Playbook Setup

Provision an application server (Apache) and a database server (MySQL) on AWS EC2 using an Ansible inventory and playbook.

## Contents

1. [Passwordless Authentication](#1-passwordless-authentication)
2. [Project Setup](#2-project-setup)
3. [Inventory](#3-inventory)
4. [Playbook](#4-playbook)
5. [Syntax Check](#5-syntax-check)
6. [Run the Playbook](#6-run-the-playbook)
7. [Verify Servers](#7-verify-servers)
8. [Example: Deploy a Web Page](#8-example-deploy-a-web-page)
9. [Key Concept](#key-concept)

---

## 1. Passwordless Authentication

### Option A: SSH Public Key (recommended)

Copy the WSL public key to the EC2 instance using the PEM file:

```bash
ssh-copy-id -f "-o IdentityFile ./ansible.pem" ubuntu@<EC2-PUBLIC-IP>
```

Test the connection:

```bash
ssh ubuntu@<EC2-PUBLIC-IP>
```

### Option B: Password Authentication

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

Set:

```text
PasswordAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

## 2. Project Setup

```bash
mkdir ansible
cd ansible
```

```text
ansible/
├── inventory.ini
└── playbook.yml
```

## 3. Inventory

Create `inventory.ini` with separate groups for application and database servers:

```ini
[app]
<APP-SERVER-PUBLIC-IP>

[db]
<DB-SERVER-PUBLIC-IP>
```

Test connectivity:

```bash
# All servers
ansible -i inventory.ini all -m ping -u ubuntu

# Application servers only
ansible -i inventory.ini app -m ping -u ubuntu

# Database servers only
ansible -i inventory.ini db -m ping -u ubuntu
```

## 4. Playbook

Create `playbook.yml`:

```yaml
---
- hosts: app
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: yes

- hosts: db
  become: true

  tasks:
    - name: Install MySQL
      ansible.builtin.apt:
        name: mysql-server
        state: present
        update_cache: yes
```

Structure:

```text
Playbook
├── Play 1 → app servers → Install Apache
└── Play 2 → db servers  → Install MySQL
```

## 5. Syntax Check

```bash
ansible-playbook -i inventory.ini playbook.yml --syntax-check
```

Expected output:

```text
playbook: playbook.yml
```

## 6. Run the Playbook

```bash
# All plays
ansible-playbook -i inventory.ini playbook.yml

# Application play only
ansible-playbook -i inventory.ini playbook.yml --limit app

# Database play only
ansible-playbook -i inventory.ini playbook.yml --limit db
```

## 7. Verify Servers

Apache:

```bash
ansible app -i inventory.ini -m shell -a "systemctl status apache2 --no-pager"
```

MySQL:

```bash
ansible db -i inventory.ini -m shell -a "systemctl status mysql --no-pager"
```

## 8. Example: Deploy a Web Page

Install Apache on the application servers and copy a custom `index.html` to the web root.

```text
simple-playbook/
├── inventory.ini
├── playbook.yml
└── index.html
```

`inventory.ini`:

```ini
[db]
ubuntu@<DB-SERVER-PUBLIC-IP>

[app]
ubuntu@<APP-SERVER-PUBLIC-IP>
```

`playbook.yml`:

```yaml
---
- hosts: app
  become: true
  tasks:
    - name: Install Apache httpd
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Copy file with owner and permissions
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html
        owner: root
        group: root
        mode: "0644"
```

Run:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Verify by opening `http://<APP-SERVER-PUBLIC-IP>` in a browser.

## Key Concept

```text
inventory.ini
      │
      ├── [app] → Application EC2
      │
      └── [db]  → Database EC2
              │
              ▼
        playbook.yml
              │
       ┌──────┴──────┐
       ▼             ▼
    app play       db play
       │             │
     Apache         MySQL
```
