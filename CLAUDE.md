# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ansible playbooks for provisioning Debian servers. Handles user setup, SSH hardening, Docker, Traefik reverse proxy, Cloudflare Tunnel, and app deployment via Docker Compose.

## Commands

```bash
# First run (as root, creates user and hardens SSH)
ansible-playbook -i inventory.yml debian.yml --user root --limit <host>

# Provision server (base OS only)
ansible-playbook -i inventory.yml debian.yml

# Set up infrastructure (Docker + Traefik)
ansible-playbook -i inventory.yml infra.yml

# Deploy app (cloudflare-tunnel + app)
ansible-playbook -i inventory.yml apps.yml --limit <host>

# Full setup + deploy
ansible-playbook -i inventory.yml playbook.yml

# Edit encrypted secrets
ansible-vault edit group_vars/vault.yml
```

## Architecture

Three-phase playbook design:

- `debian.yml` - base OS setup: `base`
- `infra.yml` - infrastructure: `docker` -> `traefik`
- `apps.yml` - app deployment: `cloudflare-tunnel` -> `app`
- `playbook.yml` - runs all three in sequence

Each host in `inventory.yml` represents one app deployment with per-host vars (`app_name`, `app_domain`, `app_repo`). Shared defaults live in `group_vars/all.yml`. Secrets are in `group_vars/vault.yml` (ansible-vault encrypted, password in `.vault_pass`).

## Current State

Only the `base` role has tasks implemented. The `docker`, `traefik`, `cloudflare-tunnel`, and `app` role directories exist but have no tasks yet.

## Conventions

- All playbooks target the `webservers` host group with `become: true`
- Secrets go in `group_vars/vault.yml`, referenced as `vault_` prefixed vars
- The `base` role copies `~/.ssh/id_ed25519.pub` from the local machine
- Use `--limit <host>` to target a single server
