# Cisco IOS-XE config & upgrade automation

## Setup
```bash
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
cp group_vars/cyber-vision-switches/vault.yml.example group_vars/cyber-vision-switches/vault.yml
ansible-vault encrypt group_vars/cyber-vision-switches/vault.yml
```
Edit `inventory.yml`, `group_vars/cyber-vision-switches/main.yml`, and `host_vars/*.yml`.
Add `--ask-vault-pass` to each command below.

## Usage
```bash
ansible-playbook cyber-vision-playbooks/config.yml --check --diff --ask-vault-pass  # dry run
ansible-playbook cyber-vision-playbooks/config.yml --ask-vault-pass                 # apply
ansible-playbook cyber-vision-playbooks/upgrade.yml --ask-vault-pass                # upgrade, one switch at a time
ansible-playbook cyber-vision-playbooks/sdflash_prep.yml -e confirm_format=true --ask-vault-pass  # one-time, destructive
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
