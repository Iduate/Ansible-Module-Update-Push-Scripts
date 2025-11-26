# Ansible Module Update & Push Scripts
Complete automation solution for selective module updates in production and module pushes in non-production environments.

## Project Structure

```
MODULE UPDATE/
├── production/
│   └── update_module.yml          # Production module update playbook
├── non-production/
│   └── push_module.yml            # Non-production module push playbook
├── roles/
│   ├── module_update/
│   │   └── tasks/
│   │       └── main.yml
│   └── module_push/
│       └── tasks/
│           └── main.yml
├── inventory/
│   └── hosts.ini                  # Ansible inventory (servers)
├── ansible.cfg                    # Ansible configuration
└── README.md                       # This file
```

---

## PRODUCTION: Module Update Playbook

### Objective
Automate selective module updates from the main branch without redeploying the entire codebase.

### Key Features
- ✅ Pull latest module version from main branch
- ✅ Automatic backup of current module before replacement
- ✅ Rollback capability if update fails
- ✅ Optional service restart
- ✅ Comprehensive error handling and logging

### Usage

```bash
ansible-playbook production/update_module.yml -i inventory/hosts.ini --limit production_servers
```

### Interactive Prompts
1. **Module name**: Name of the module to update (e.g., `auth_module`, `payment_service`)
2. **Restart service**: Whether to restart the service after update (yes/no)

### Workflow
1. Validates module name
2. Creates timestamped backup of current module
3. Clones repository to temporary location
4. Verifies module exists in repository
5. Replaces old module with new version
6. Sets proper permissions and ownership
7. Optionally restarts related service
8. Cleans up temporary files
9. Provides status report with backup location

### Example Run
```bash
$ ansible-playbook production/update_module.yml -i inventory/hosts.ini

Enter the module name to update: auth_module
Enter the restart service after update? (yes/no) [no]: yes

# Output will show:
# - Backup created at: /opt/backups/auth_module_backup_20240115T120000
# - Module successfully updated
# - Service restarted
```

### Backup Location
Backups are stored in: `/opt/backups/`
Format: `{module_name}_backup_{timestamp}`

### Rollback
If the update fails, the module is automatically restored from the backup. You can also manually restore:

```bash
cp -r /opt/backups/auth_module_backup_20240115T120000/* /opt/project/auth_module/
```

### Variables to Configure
Edit the `vars` section in `production/update_module.yml`:

```yaml
project_repo_url: "https://github.com/your-org/your-project.git"
project_local_path: "/opt/project"
backup_base_path: "/opt/backups"
```

---

## NON-PRODUCTION: Module Push Playbook

### Objective
Enable developers to commit and push module-level changes to automatically generated feature branches.

### Key Features
- ✅ Automatic feature branch creation (prevents direct main/dev pushes)
- ✅ Module-level staging and committing
- ✅ Automatic timestamped branch naming
- ✅ Remote verification
- ✅ Comprehensive error handling

### Usage

```bash
ansible-playbook non-production/push_module.yml -i inventory/hosts.ini --limit non_production_servers
```

### Interactive Prompts
1. **Module name**: Name of the module to push (e.g., `auth_module`, `payment_service`)
2. **Commit message**: Custom commit message for changes

### Workflow
1. Validates module name and commit message
2. Checks git status of module
3. Stages module changes
4. Generates feature branch name: `feature/{module_name}_{timestamp}`
5. Creates new feature branch locally
6. Commits changes with provided message
7. Pushes to remote repository
8. Verifies remote branch creation
9. Provides pull request guidance

### Example Run
```bash
$ ansible-playbook non-production/push_module.yml -i inventory/hosts.ini

Enter the module name to push: auth_module
Enter commit message: Add SSO integration

# Output will show:
# - Feature branch: feature/auth_module_20240115_120000
# - Changes staged and committed
# - Pushed to remote successfully
# - PR can be created at: GitHub/compare/feature/auth_module_20240115_120000
```

### Branch Naming Convention
`feature/{module_name}_{YYYYMMDD_HHMMSS}`

Example: `feature/auth_module_20240115_120000`

### Safety Features
- ❌ Prevents direct pushes to `main` branch
- ❌ Prevents direct pushes to `dev` branch
- ✅ Forces creation of feature branch
- ✅ Validates branch exists on remote before completion

### Variables to Configure
Edit the `vars` section in `non-production/push_module.yml`:

```yaml
project_repo_url: "https://github.com/your-org/your-project.git"
project_local_path: "/home/developer/project"
feature_branch_prefix: "feature"
```

### Next Steps After Push
1. Go to GitHub repository
2. Create Pull Request from your feature branch
3. Add description and request reviewers
4. Wait for code review and approval
5. Merge to appropriate branch (dev/staging)

---

## Prerequisites

### System Requirements
- **Ansible**: Version 2.9 or higher
- **Git**: Installed on all target servers
- **SSH Access**: Key-based authentication configured
- **Python**: 3.6+ on target servers

### Installation

#### Install Ansible
```bash
# On macOS
brew install ansible

# On Ubuntu/Debian
sudo apt-get update
sudo apt-get install ansible

# On CentOS/RHEL
sudo yum install ansible

# Using pip (any OS)
pip install ansible>=2.9
```

#### Verify Installation
```bash
ansible --version
ansible-inventory --list -i inventory/hosts.ini
```

---

## Configuration

### 1. Update Inventory File
Edit `inventory/hosts.ini` with your servers:

```ini
[production_servers]
prod-1 ansible_host=192.168.1.10 ansible_user=ansible

[non_production_servers]
dev-1 ansible_host=192.168.2.10 ansible_user=developer
```

### 2. Setup SSH Keys
```bash
# Generate keys (if not already done)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/prod_key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/dev_key

# Copy public keys to servers
ssh-copy-id -i ~/.ssh/prod_key.pub ansible@prod-server-1
ssh-copy-id -i ~/.ssh/dev_key.pub developer@dev-server-1
```

### 3. Configure Ansible Variables
Edit playbook variables:
- **project_repo_url**: Your GitHub repository URL
- **project_local_path**: Where your project is deployed
- **backup_base_path**: Where to store module backups (production only)

### 4. Test Connectivity
```bash
ansible all -i inventory/hosts.ini -m ping
```

---

## Usage Examples

### Production: Update Auth Module
```bash
ansible-playbook production/update_module.yml \
  -i inventory/hosts.ini \
  --limit prod-server-1 \
  -e "module_name=auth_module" \
  -e "restart_service=yes"
```

### Production: Update Multiple Servers
```bash
ansible-playbook production/update_module.yml \
  -i inventory/hosts.ini \
  --limit production_servers
```

### Non-Production: Push Payment Module
```bash
ansible-playbook non-production/push_module.yml \
  -i inventory/hosts.ini \
  --limit dev-server-1 \
  -e "module_name=payment_service" \
  -e "commit_message=Fix payment gateway bug"
```

### Enable Verbose Logging
```bash
ansible-playbook production/update_module.yml \
  -i inventory/hosts.ini \
  -vvv  # Very verbose (debug level)
```

### Dry Run (Check Mode)
```bash
ansible-playbook production/update_module.yml \
  -i inventory/hosts.ini \
  --check  # Shows what would happen without making changes
```

---

## Troubleshooting

### SSH Connection Issues
```bash
# Test SSH connectivity
ssh -i ~/.ssh/prod_key ansible@prod-server-1

# Check SSH key permissions
chmod 600 ~/.ssh/prod_key
chmod 644 ~/.ssh/prod_key.pub
```

### Git/Repository Issues
```bash
# Verify Git access
ssh -i ~/.ssh/prod_key -T git@github.com

# Check repository URL
cd /opt/project && git remote -v
```

### Permission Denied Errors
```bash
# Grant execute permissions
ansible production_servers -i inventory/hosts.ini -m file \
  -a "path=/opt/project mode=0755 recurse=yes" \
  -b  # Use sudo
```

### Module Not Found
```bash
# Verify module exists in repository
git ls-tree -r --name-only main | grep {module_name}
```

### Backup Issues
```bash
# Check backup directory
ansible production_servers -i inventory/hosts.ini -m shell \
  -a "ls -la /opt/backups/" \
  -b
```

---

## Best Practices

### Production Deployments
1. ✅ Always run in check mode first (`--check`)
2. ✅ Test on a staging server before production
3. ✅ Schedule updates during maintenance windows
4. ✅ Keep backup retention policy (e.g., keep last 10 backups)
5. ✅ Monitor service logs after update
6. ✅ Have rollback plan ready

### Development Workflows
1. ✅ Always commit your changes locally first
2. ✅ Use descriptive commit messages
3. ✅ Create feature branches for all changes
4. ✅ Request code review before merging
5. ✅ Delete feature branch after merge
6. ✅ Keep feature branches short-lived (< 1 week)

### Repository Management
1. ✅ Keep main branch always deployable
2. ✅ Use proper branching strategy (Git Flow)
3. ✅ Require PR reviews before merge
4. ✅ Maintain clear commit history
5. ✅ Tag releases with version numbers

---

## Advanced Configuration

### Custom Service Restart
Modify the service restart section to match your service names:

```yaml
- name: Restart the service (Optional)
  systemd:
    name: "{{ module_name }}_service"  # Change to your service name
    state: restarted
  become: yes
  when: restart_service | lower == "yes"
```

### Add Pre/Post Update Hooks
```yaml
- name: Run pre-update validation
  shell: /opt/project/scripts/validate.sh {{ module_name }}

- name: Run post-update tests
  shell: /opt/project/scripts/test.sh {{ module_name }}
```

### Email Notifications
```yaml
- name: Send update notification
  mail:
    host: smtp.example.com
    port: 25
    to: devops@example.com
    subject: "Module {{ module_name }} updated"
    body: "Update completed at {{ ansible_date_time.iso8601 }}"
```

---

## Security Considerations

1. **SSH Keys**: Use key-based authentication, never passwords
2. **GitHub Tokens**: Use SSH keys or deploy keys for Git access
3. **Backups**: Store backups in secure location with proper permissions
4. **Access Control**: Limit who can run these playbooks
5. **Logging**: Keep audit logs of all deployments
6. **Validation**: Always verify changes before applying

---

## Support & Maintenance

### Regular Tasks
- Review and update playbooks quarterly
- Test backup/restore procedures monthly
- Rotate SSH keys annually
- Update inventory as infrastructure changes
- Archive old backups periodically

### Monitoring
```bash
# Watch playbook logs
tail -f logs/ansible.log

# Check backup sizes
du -sh /opt/backups/*

# Monitor service status after update
systemctl status {module_name}_service
```

---

## Version History

**v1.0.0** - Initial release
- Production module update playbook
- Non-production module push playbook
- Backup and rollback functionality
- Comprehensive error handling

---

## License & Support
For issues or improvements, please contact your DevOps team.

---

## Quick Reference

### Production Commands
```bash
# Update module with service restart
ansible-playbook production/update_module.yml -i inventory/hosts.ini

# Update specific server
ansible-playbook production/update_module.yml -i inventory/hosts.ini --limit prod-1
```

### Non-Production Commands
```bash
# Push module changes
ansible-playbook non-production/push_module.yml -i inventory/hosts.ini

# Push to specific server
ansible-playbook non-production/push_module.yml -i inventory/hosts.ini --limit dev-1
```

### Utility Commands
```bash
# Test inventory
ansible-inventory -i inventory/hosts.ini --list

# Test connectivity
ansible all -i inventory/hosts.ini -m ping

# List backups
ansible production_servers -i inventory/hosts.ini -m shell -a "ls -la /opt/backups/" -b
```
