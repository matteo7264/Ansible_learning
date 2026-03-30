# Ansible — Learning Project

> My first hands-on experience with Ansible: automating the deployment and configuration of an Nginx web server on an Ubuntu Server virtual machine.

---

## Context

This is a **beginner project** built while learning Ansible from scratch. The goal was never to build something complex — it was to understand how Ansible works, how to write playbooks, and how to automate infrastructure tasks in a clean, repeatable way.


---

## Goal

- Understand the core concepts of Ansible (inventory, modules, playbooks, roles, tags)
- Automate the installation and configuration of **Nginx** on a remote Ubuntu Server
- Deploy a custom HTML page as the default Nginx site
- Organize the project using **Ansible Roles**
- Practice writing cross-distribution playbooks using the `package` module

---

## Architecture

```
Control Node                          Managed Node
┌──────────────────────────┐          ┌─────────────────────────┐
│  Windows 11 + WSL        │          │  Ubuntu Server (VM)     │
│  (Ubuntu via WSL)        │   SSH    │                         │
│                          │ ──────►  │  - Nginx                │
│  - Ansible               │   Key    │  - Custom HTML page     │
│  - Playbooks / Roles     │  Based   │                         │
│  - inventory.ini         │          │                         │
└──────────────────────────┘          └─────────────────────────┘
```

**Communication:** Ansible connects to the managed node over SSH using key-based authentication (ED25519). No agent is required on the server side.

---

## Project Structure

```
Ansible_learning/
├── inventory.ini               # Defines the managed hosts
├── site.yml           # Main playbook (Nginx install + config)
└── roles/
    └── nginx/
        ├── tasks/
        │   └── main.yml        # Core tasks (install, copy, start)
        ├── handlers/
        │   └── main.yml        # Handlers (e.g. restart Nginx on change)
        └── files/
            └── default_site.html  # Custom HTML page deployed to the server

```

---

## Prerequisites

| Requirement | Details |
|---|---|
| OS (Control Node) | Windows 11 with WSL2 (Ubuntu) |
| Ansible | `sudo apt install -y ansible` |
| SSH access | Key-based auth configured on the managed node |
| Target server | Ubuntu Server (VM or physical) |

---

## How to Use

### 1. Clone the repository

```bash
git clone https://github.com/matteo7264/Ansible.git
cd Ansible
```

### 2. Edit the inventory

Update `inventory.ini` with your server's IP address and username:

```ini
[servers]
<server_ip> ansible_user=<your_user> ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

### 3. Run the playbook

```bash
ansible-playbook -i inventory.ini install_nginx.yml --ask-become-pass
```

> The `--ask-become-pass` flag will prompt for the **sudo password** of the remote user (not root).

### 4. Run specific tasks with tags

```bash
# Install Nginx only
ansible-playbook --tags nginx -i inventory.ini install_nginx.yml --ask-become-pass

# Deploy the HTML file only
ansible-playbook --tags copy -i inventory.ini install_nginx.yml --ask-become-pass
```

### 5. Verify

Open a browser and navigate to `http://<server_ip>` — you should see the custom HTML page served by Nginx.

---

## What I Learned

Working through this project taught me the following Ansible concepts:

- **Inventory files** — how to define and group managed hosts, and pass per-host variables like `ansible_user`
- **Ad-hoc commands** — running one-off tasks directly from the CLI without a playbook
- **Playbooks** — writing YAML-based automation files with multiple tasks
- **Idempotency** — Ansible only makes changes when needed; running a playbook twice is safe
- **Modules** — `apt`, `package`, `copy`, `service`, and when to use each
- **The `package` module** — cross-distribution alternative to `apt`/`yum` that removes the need for `when` conditions
- **Tags** — selectively running parts of a playbook without executing everything
- **Handlers** — triggering actions (like restarting a service) only when a change actually occurs
- **Roles** — structuring a project into reusable, modular components

---

## Next Steps

- [ ] Add a Jinja2 template for the Nginx configuration file
- [ ] Use `ansible-vault` to encrypt sensitive variables
- [ ] Add a second managed node (e.g., CentOS/Rocky Linux)
- [ ] Explore Ansible Galaxy to use community roles

---

## Notes

- This project was built as a **learning exercise** — the playbooks are intentionally simple and well-commented
- The Nginx role uses the `package` module for cross-distribution compatibility, even though the only current target is Ubuntu
- If Nginx fails to start, check whether another service (e.g., Apache) is already using port 80:
  ```bash
  sudo systemctl stop apache2 && sudo systemctl disable apache2
  ```

---

*Built while learning Ansible — feedback and suggestions always welcome.*
