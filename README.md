# Hetzner Storage Box Automated Restic Backup (Ansible)

This project provides an Ansible-based solution to automate secure backups from your server(s) to a Hetzner Storage Box using [restic](https://restic.net/). It includes notification support via Gotify.

## Features
- Automated restic backup to Hetzner Storage Box via SFTP
- SSH key-based authentication (Password Login is not supported)
- Automatic repository initialization and remote directory creation
- Configurable backup sources and retention policy
- Gotify notifications for backup status and errors
- Cron-based daily backup scheduling
- Log rotation for backup logs

## Prerequisites
- Linux server(s) with Python 3.x
- Ansible 2.9+
- Hetzner Storage Box with SSH/SFTP enabled
- SSH key for Storage Box access
- (Optional) Gotify server for notifications

## Quick Start

1. **Clone this repository:**
   ```bash
   git clone <repository-url>
   cd hetzner_backup
   ```

2. **Copy and edit configuration:**
   ```bash
   cp group_vars/all.yml.sample group_vars/all.yml
   # Edit group_vars/all.yml with your Storage Box credentials and backup paths
   ```
   - Make sure `restic_repo` uses a writable path, e.g.:
     ```yaml
     restic_repo: "sftp:<username>@<your-storagebox>.de:backups/restic/{{ inventory_hostname }}"
     # Do NOT use a leading slash unless it is /home/<username>/...
     ```
   - Set `restic_password` 
   - Adjust `backup_sources`, retention, and Gotify settings as needed

3. **Configure your inventory:**
   ```bash
   cp hosts.ini.sample hosts.ini
   # Edit hosts.ini with your server(s)
   ```

4. **Run the playbook:**
   ```bash
   ansible-playbook -i hosts.ini playbook.yml
   ```

## How It Works
- Installs restic, jq, and curl
- Sets up `/etc/restic.env` with all required environment variables
- Deploys a backup script (`/usr/local/bin/restic-backup.sh`) that:
  - Ensures the remote directory exists
  - Initializes the restic repository if needed
  - Runs backups for all configured sources
  - Prunes old backups according to retention policy
  - Sends Gotify notifications on success/failure
- Adds a cron job for daily backups at 02:00
- Manages SSH keys and known_hosts for secure, non-interactive SFTP
- Rotates backup logs via logrotate

## Security Notes
- SSH keys are used for authentication; passwords are not required
- Only paths inside your Storage Box home directory are writable (e.g. `backups/...`)
- Use Ansible Vault for sensitive variables like `restic_password`

## Restore from Storage Box
To restore files, use restic or sftp:
```bash
restic -r sftp:<username>@<your-storagebox>.de:backups/restic/<hostname> snapshots
restic -r sftp:<username>@<your-storagebox>.de:backups/restic/<hostname> restore latest --target /restore/path
```

## Manual Restic Operations

For manual interaction with your restic repository, SSH into the target server and use the following commands:

### Setup
```bash
# SSH into your target server
ssh your-server

# Load the restic environment variables (required for all commands)
# This has to be done as superuser (`sudo su`)
source /etc/restic.env
```

### View Snapshots
```bash
# List all snapshots
restic snapshots
```

### View Backup Contents
```bash
# List files in the latest snapshot
restic ls latest

# List files in a specific snapshot
restic ls <snapshot-id>
```

### Repository Management
```bash
# Check repository integrity
restic check

# View repository statistics
restic stats

# View statistics for a specific snapshot
restic stats <snapshot-id>
```

### Manual Backup
```bash
# Run a manual backup of all configured sources
/usr/local/bin/restic-backup.sh

# Or backup a specific directory manually
source /etc/restic.env
restic backup /path/to/directory
```


## References
- [Hetzner Storage Box Docs](https://docs.hetzner.com/de/storage/storage-box/access/access-ssh-rsync-borg#restic)
- [restic Documentation](https://restic.readthedocs.io/)
- [Gotify Documentation](https://gotify.net/docs/)

## License
MIT
