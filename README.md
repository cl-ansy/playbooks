# Playbooks

Ansible playbooks for provisioning Debian servers. Handles user setup, SSH hardening, Docker, Traefik, Cloudflare Tunnel, and app deployment.

## Requirements

- Ansible installed locally (`sudo dnf install ansible`)
- SSH access to the target server (root for first run)
- A Debian-based server (tested on Debian 13)

## Structure

```
playbook.yml               # runs debian + infra + apps
debian.yml                 # base OS setup
infra.yml                  # docker + traefik
apps.yml                   # cloudflare-tunnel + app
inventory.yml              # hosts and per-app config
group_vars/
  all.yml                  # shared defaults
  vault.yml                # encrypted secrets
roles/
  base/                    # user creation, SSH hardening
  docker/                  # Docker + Compose
  traefik/                 # Traefik reverse proxy + TLS
  cloudflare-tunnel/       # cloudflared service
  app/                     # app containers via docker-compose
```

## Setup

### 1. Set your vault password

The vault password unlocks the encrypted secrets file. Edit `.vault_pass` with a strong password:

```bash
echo 'your-vault-password' > .vault_pass
chmod 600 .vault_pass
```

This file is gitignored.

### 2. Encrypt the vault and set your sudo password

```bash
ansible-vault encrypt group_vars/vault.yml
ansible-vault edit group_vars/vault.yml
```

Set `vault_user_password` to the sudo password you want on the server.

## Usage

### First run (as root, sets up user and SSH)

```bash
ansible-playbook -i inventory.yml debian.yml --user root --limit <host>
```

### Provision a server (base OS only)

```bash
ansible-playbook -i inventory.yml debian.yml```

### Set up infrastructure (Docker + Traefik)

```bash
ansible-playbook -i inventory.yml infra.yml```

### Deploy an app

```bash
ansible-playbook -i inventory.yml apps.yml --limit arbolio```

### Full setup + deploy

```bash
ansible-playbook -i inventory.yml playbook.yml```

### Target a single host

```bash
ansible-playbook -i inventory.yml playbook.yml --limit arbolio```

## Adding a new project

Add a host entry to `inventory.yml` under `webservers`:

```yaml
my-project:
  ansible_host: 1.2.3.4
  app_name: my-project
  app_domain: my-project.com
  app_repo: git@github.com:user/my-project.git
```

The base/docker/traefik roles apply automatically. The `app` role uses these variables to template the deployment.
