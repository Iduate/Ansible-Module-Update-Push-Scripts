# Ansible Module Update & Push Scripts

Automate selective module updates in production and module pushes in non-production environments.

---

## Quick Start

### 1. Installation
```bash
pip install -r requirements.txt
```

### 2. Configuration
Edit `inventory/hosts.ini` with your servers:
```ini
[production_servers]
prod-1 ansible_host=192.168.1.10 ansible_user=ansible

[non_production_servers]
dev-1 ansible_host=192.168.2.10 ansible_user=developer
```

Update variables in `group_vars/all.yml`:
```yaml
git_config:
  repo_url: "https://github.com/your-org/your-project.git"
```

### 3. Test Connectivity
```bash
ansible all -i inventory/hosts.ini -m ping
```

---

## Usage

### Production: Update Module
```bash
ansible-playbook production/update_module.yml -i inventory/hosts.ini
```
- Prompts for module name
- Creates automatic backup
- Pulls from main branch
- Optional service restart
- Auto-rollback on failure

### Non-Production: Push Module
```bash
ansible-playbook non-production/push_module.yml -i inventory/hosts.ini
```
- Auto feature branch creation
- Prevents main/dev direct pushes
- Automatic commit and push
- Ready for PR

### Using Helper Scripts
```bash
./run.sh          # Linux/macOS
.\run.ps1         # Windows
```

---

## File Structure

```
├── production/update_module.yml       # Production playbook
├── non-production/push_module.yml     # Non-production playbook
├── inventory/hosts.ini                # Server inventory
├── group_vars/all.yml                 # Global variables
├── ansible.cfg                        # Ansible config
├── run.sh                             # Bash helper
├── run.ps1                            # PowerShell helper
└── requirements.txt                   # Dependencies
```

---

## Features

### Production Update
✅ Automatic backup before update
✅ Pull from main branch
✅ Safe module replacement
✅ Rollback on failure
✅ Optional service restart
✅ Comprehensive logging

### Non-Production Push
✅ Auto feature branch creation
✅ Branch protection (no main/dev direct push)
✅ Timestamped branch naming
✅ Remote verification
✅ PR guidance

---

## Configuration

### Edit Variables
```bash
# Repository URL
group_vars/all.yml → git_config.repo_url

# Project paths
production/update_module.yml → project_local_path
non-production/push_module.yml → project_local_path

# Backup location
production/update_module.yml → backup_base_path
```

---

## Examples

### Update auth_module with service restart
```bash
ansible-playbook production/update_module.yml \
  -i inventory/hosts.ini \
  -e "module_name=auth_module" \
  -e "restart_service=yes"
```

### Push payment_service changes
```bash
ansible-playbook non-production/push_module.yml \
  -i inventory/hosts.ini \
  -e "module_name=payment_service" \
  -e "commit_message=Add Stripe integration"
```

### Dry run (check mode)
```bash
ansible-playbook production/update_module.yml --check
```

---

## Backup Management

Backups are stored in: `/opt/backups/`

### List backups
```bash
ansible production_servers -i inventory/hosts.ini \
  -m shell -a "ls -la /opt/backups/" -b
```

### Manual rollback
```bash
cp -r /opt/backups/module_name_backup_TIMESTAMP/* /opt/project/module_name/
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| SSH connection refused | Check SSH key: `chmod 600 ~/.ssh/key` |
| Module not found | Verify in main branch: `git ls-tree -r --name-only main` |
| Permission denied | Use sudo: add `-b` flag |
| Git push failed | Check git access and SSH keys |
| Service not restarting | Verify service name in playbook |

---

## Requirements

- Ansible 2.9+
- Python 3.6+
- Git installed
- SSH key-based authentication
- Linux/macOS or Windows PowerShell

---

## Security Notes

✅ SSH key-based auth only (no passwords)
✅ Automatic backups before changes
✅ Automatic rollback on failure
✅ Feature branch protection
✅ Audit logging available

---

## Support

For issues:
1. Check troubleshooting section above
2. Run with verbose: `ansible-playbook playbook.yml -vvv`
3. Check logs in `logs/` directory
4. Review playbook variables

---

## License

Use freely for your infrastructure automation needs.
