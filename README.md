# Cisco IOS-XE config & upgrade automation

## Setup
```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
cp group_vars/switches/vault.yml.example group_vars/switches/vault.yml
ansible-vault encrypt group_vars/switches/vault.yml
```
Edit `inventory.yml`, `group_vars/switches/main.yml`, and `host_vars/*.yml`.
Add `--ask-vault-pass` to each command below.

## Usage
```bash
ansible-playbook playbooks/config.yml --check --diff   # dry run
ansible-playbook playbooks/config.yml                  # apply
ansible-playbook playbooks/upgrade.yml                 # upgrade, one switch at a time
ansible-playbook playbooks/sdflash_prep.yml -e confirm_format=true   # one-time, destructive
```

## Notes
- `version 17.18.03` is the upgrade target (`target_version`), not config.
- `format sdflash:` / `partition sdflash: iox` are exec commands -> `sdflash_prep.yml`.
- `domain cisco.com` is written as `ip domain name cisco.com`.
- DNS: set `dns_server` (and optionally `dns_server_alt`) in `main.yml`; renders `ip name-server <primary> [<alternate>]`. Override per host in `host_vars` if needed.
- The admin user is managed via `ios_user` so the hashed secret does not cause a diff each run.
- Upgrade pushes the image via SCP (`ip scp server enable` is applied automatically).
- Typical order: sdflash_prep -> upgrade -> config.
- To add routers later, create a `routers` group with its own vars/template.
