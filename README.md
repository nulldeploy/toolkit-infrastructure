# ⚙️ toolkit-infra

Ansible automation for deploying [toolkit](https://github.com/nulldeploy/toolkit) to a clean VPS. A single playbook run sets up the entire stack from scratch.

---

## What the Playbook Does

```
Clean VPS (Ubuntu)
    │
    ├── UFW: allow 22, 80, 443, 9100 / deny everything else
    ├── Docker: install via official script
    ├── User devops: sudo + docker groups, SSH key
    ├── toolkit: git clone → .env → systemd service → start
    ├── Health check: GET /health → 200 OK
    ├── Node Exporter: download → binary → systemd service
    └── Done
```

---

## Stack

| Tool            | Version   |
| --------------- | --------- |
| Ansible         | 2.15+     |
| Ubuntu (target) | 22.04 LTS |
| Node Exporter   | 1.10.2    |

---

## Structure

```
toolkit-infra/
├── playbook.yml            # main playbook
├── inventory.ini           # hosts (not in git)
├── templates/
│   ├── env.j2              # .env template for the application
│   ├── service.j2          # systemd unit template for toolkit
│   └── node_exporter.j2   # systemd unit template for node_exporter
└── vars/
    ├── secrets.yml         # encrypted secrets (not in git)
    └── secrets.example.yml # secrets structure example
```

---

## Quick Start

### 1. Dependencies

```bash
pip install ansible
ansible-galaxy collection install community.general
```

### 2. Inventory

```bash
cp inventory.example.ini inventory.ini
# fill in your VPS IP and SSH key path
```

```ini
[vps]
toolkit ansible_host=YOUR_VPS_IP ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

### 3. Secrets

```bash
cp vars/secrets.example.yml vars/secrets.yml
ansible-vault encrypt vars/secrets.yml
# set the devops user password
```

```yaml
# secrets.yml
app_usr_passwd: "your_password"
```

### 4. Run

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

---

## Idempotency

The playbook is safe to run multiple times — repeated runs won't break a running deployment:

- `unarchive` with `creates:` — skips if binary already exists
- `shell` with `creates:` — skips Docker install if already installed
- `git` with `update: yes` — pulls changes without recreating the repo
- `systemd` — idempotent by design
- `user`, `ufw`, `authorized_key` — idempotent Ansible modules

---

## Variables

| Variable             | Description                           | Default                                     |
| -------------------- | ------------------------------------- | ------------------------------------------- |
| `app_user`           | System user for the application       | `devops`                                    |
| `app_dir`            | Repository path on the server         | `/home/devops/toolkit`                      |
| `repo`               | Git repository URL                    | `https://github.com/nulldeploy/toolkit.git` |
| `app_port`           | Flask application port                | `5000`                                      |
| `node_exporter_user` | User for Node Exporter                | `node_exporter`                             |
| `app_usr_passwd`     | devops user password (in secrets.yml) | —                                           |

---

## Related Repositories

- [toolkit](https://github.com/nulldeploy/toolkit) — main application
- [toolkit-monitoring](https://github.com/nulldeploy/toolkit-monitoring) — monitoring stack
