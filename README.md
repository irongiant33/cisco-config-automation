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

Pre-requisite for running the below commands is that each device in the `inventory.yml` is reachable over IP and each device has a SSH password stored in the `vault.yml` file.

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

## RSPAN Utilization Checks

`show interfaces <interface-id> | include line protocol|input rate|output rate`
 - ensure 5 minute output rate does not consistently exceed 70-80% of the port's total capacity

`show interfaces <interface-id> counters errors`
- If the drop counter is actively incrementing, your RSPAN traffic is saturating the port.
- `clear counters <interface-id>` and try again to reassess

To mitigate traffic:
- Filter the Traffic: Instead of monitoring all traffic, use a VACL (VLAN Access Control List) to filter out heavy, irrelevant protocols (like backup traffic, storage traffic, or video streaming) before it enters the RSPAN VLAN.
- Restrict the Direction: Monitor only rx (receive) or tx (transmit) traffic on the source ports rather than both, effectively cutting the potential mirrored bandwidth in half.
