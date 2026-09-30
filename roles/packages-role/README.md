# Ansible Packages Role

Reusable **Ansible Role** for provisioning Ubuntu servers with common system packages, Docker, Docker Compose, and Nginx.

## 🚀 Features

* Update APT package cache
* Install common packages
* Install and configure Docker
* Install Docker Compose
* Add user to the `docker` group
* Verify Docker service and access
* Remove Apache to prevent port `80` conflicts
* Install and enable Nginx
* Use Ansible handlers for Docker restart
* Idempotent server configuration

## 📁 Project Structure

```text
.
├── packages/
│   ├── defaults/
│   │   └── main.yml
│   ├── handlers/
│   │   └── main.yml
│   ├── tasks/
│   │   └── main.yml
│   └── vars/
│       └── main.yml
├── inventory.ini
├── playbook.yml
└── README.md
```

## ⚙️ Requirements

* Ansible
* Ubuntu server
* SSH access
* Sudo privileges

## 🔧 Configuration

### Inventory

```ini
[app]
app1 ansible_host=<SERVER_IP> ansible_user=ubuntu
```

### Variables

Example:

```yaml
COMMON_PACKAGES:
  - git
  - curl
  - wget
  - unzip
  - vim

DOCKER_PACKAGES:
  - docker.io
  - docker-compose-v2

DOCKER_SERVICE: docker
DOCKER_USER: ubuntu

NGINX_PACKAGE: nginx
NGINX_SERVICE: nginx
```

## ▶️ Usage

Test the Ansible connection:

```bash
ansible all -i inventory.ini -m ping
```

Run the role:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Example `playbook.yml`:

```yaml
---
- name: Configure application server
  hosts: app
  become: true

  roles:
    - packages
```

## 🔍 Verification

Check Docker:

```bash
docker --version
docker ps
```

Check Nginx:

```bash
systemctl status nginx
```

Test Nginx:

```bash
curl http://localhost
```

> **Note:** After adding a user to the `docker` group, start a new SSH session before running `docker ps` without `sudo`.

## 🧠 Ansible Concepts Practiced

* Ansible Roles
* Inventory
* Variables
* `become`
* `apt`
* `user`
* `service`
* `command`
* `shell`
* `register`
* `debug`
* Handlers
* `notify`
* Idempotency
* Service verification

## 🎯 Purpose

This project demonstrates how Ansible can be used to **automatically provision and configure an Ubuntu server** with the basic tools required for a DevOps environment.

## 👨‍💻 Author

**Ali Ahmad Shah**

DevOps / Cloud Enthusiast
