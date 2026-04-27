# homelab-ansible

Ansible configuration for managing LXC containers on a single-node Proxmox homelab.
Handles package updates, backups, and notifications across all containers.

For a full overview of the homelab infrastructure see the
[homelab repository](https://github.com/gigan0815/homelab).

## Requirements

- Ansible 2.9+
- Python 3
- community.general collection (`ansible-galaxy collection install community.general`)
- SSH key-based access to all managed hosts

## Repository Structure

    homelab-ansible/
    ├── inventory/
    │   └── hosts.yml
    ├── group_vars/
    │   └── all/
    │       └── vault.yml
    ├── roles/
    │   ├── update/
    │   ├── paperless_backup/
    │   ├── node_exporter/
    │   ├── prometheus/
    │   └── grafana/
    ├── playbooks/
    │   ├── update.yml
    │   ├── paperless_backup.yml
    │   ├── node_exporter.yml
    │   ├── prometheus.yml
    │   └── grafana.yml
    └── ansible.cfg

## Inventory

All managed hosts are LXC containers running on Proxmox.

| Host | IP | OS |
|------|----|----|
| paperless | 192.168.1.12 | Debian 13 |
| sabnzbd | 192.168.1.13 | Debian 13 |
| prowlarr | 192.168.1.14 | Debian 13 |
| sonarr | 192.168.1.15 | Debian 13 |
| radarr | 192.168.1.16 | Debian 13 |
| jellyfin | 192.168.1.17 | Ubuntu |
| bazarr | 192.168.1.18 | Debian 13 |
| ansible | 192.168.1.19 | Debian 13 |
| prometheus | 192.168.1.21 | Debian 13 |
| grafana | 192.168.1.22 | Debian 13 |

## Playbooks

### update.yml
Updates all containers and sends a Discord notification if any container
requires a reboot after the update.

    ansible-playbook playbooks/update.yml

### paperless_backup.yml
Exports paperless-ngx documents and syncs them to OneDrive via rclone.
Runs automatically every day at 02:00 via cron on the Ansible control node.

    ansible-playbook playbooks/paperless_backup.yml

### node_exporter.yml
Deploys Node Exporter on all containers for metrics collection.

    ansible-playbook playbooks/node_exporter.yml

### prometheus.yml
Deploys Prometheus on the monitoring container.

    ansible-playbook playbooks/prometheus.yml

### grafana.yml
Deploys Grafana on the monitoring container.

    ansible-playbook playbooks/grafana.yml

## Roles

### update
- Updates apt cache
- Upgrades all packages
- Checks if reboot is required
- Sends aggregated Discord notification if any host requires a reboot

### paperless_backup
- Installs and configures rclone
- Exports paperless-ngx documents
- Syncs export to OneDrive

### node_exporter
- Downloads and installs Node Exporter binary
- Deploys systemd service

### prometheus
- Downloads and installs Prometheus binary
- Deploys configuration with auto-discovery from Ansible inventory
- Deploys systemd service

### grafana
- Installs Grafana via apt repository
- Configures root URL
- Deploys systemd service

## Vault

Sensitive credentials are stored encrypted using Ansible Vault.

    # View encrypted file
    ansible-vault view group_vars/all/vault.yml

    # Edit encrypted file
    ansible-vault edit group_vars/all/vault.yml

Required vault variables:
- `discord_webhook_id`
- `discord_webhook_token`

## Usage

Run all updates:

    ansible-playbook playbooks/update.yml

Run paperless backup manually:

    ansible-playbook playbooks/paperless_backup.yml

Deploy monitoring stack:

    ansible-playbook playbooks/node_exporter.yml
    ansible-playbook playbooks/prometheus.yml
    ansible-playbook playbooks/grafana.yml

Test connectivity to all hosts:

    ansible all -m ping
