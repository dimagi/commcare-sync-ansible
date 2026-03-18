# CommCare Sync Ansible

Ansible playbooks and roles for deploying [CommCare Data Pipeline](https://github.com/dimagi/commcare-sync).

## Setup

```bash
source .venv/bin/activate
```

## Commands

```bash
# Lint
ansible-lint

# Run Molecule tests for a role (requires Docker)
cd roles/<role>
molecule test

# Run playbook
ansible-playbook -i inventories/<env> commcare_sync.yml
```

## Structure

- `roles/` — Ansible roles (`commcare_sync`, `django`, `nginx`, `postgres`, `redis`, `superset`)
- `inventories/` — Per-environment inventory files
- `requirements.yml` — Ansible Galaxy role/collection dependencies
- `requirements-dev.txt` — Python dev tools (`ansible-dev-tools`, `molecule-docker`)

## Molecule tests

Each role has tests in `roles/<role>/molecule/default/`. Tests use Docker (Ubuntu 24.04). Run from the role directory.
