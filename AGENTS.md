# AGENTS.md for ansible-local

Ansible project for SemaphoreUI to execute remote scripts on servers. Tech stack: Ansible, YAML, SSH, SemaphoreUI.

## Development Environment

Prerequisites:
- Python 3.9+
- Ansible 2.9+ (`pip install ansible`)
- SSH access to target servers
- SSH key configured at `~/.ssh/id_rsa` (or use `-e ssh_key=/path/to/key`)

No package manager needed - Ansible is the only runtime dependency.

## Commands & Workflows

### Validate playbooks
```bash
ansible-playbook --syntax-check playbooks/run_script.yml
```

### Dry run (check mode)
```bash
ansible-playbook playbooks/run_script.yml -e "script_path=playbooks/test.sh" -i "hostname," --check
```

### Run manually (for testing)
```bash
ansible-playbook playbooks/run_script.yml \
  -i "server1.example.com," \
  -e "script_path=playbooks/test.sh" \
  --user ubuntu \
  --private-key ~/.ssh/id_rsa
```

### List all playbooks
```bash
ls -la playbooks/
```

## Role

You maintain and extend this Ansible project. You create playbooks that SemaphoreUI executes against remote servers. You do NOT manage the SemaphoreUI installation itself - only the playbooks and configuration it runs.

## Scope of Responsibility

You modify:
- `playbooks/*.yml` - Ansible playbooks
- `ansible.cfg` - Ansible configuration
- Scripts in `playbooks/` that get transferred to remote hosts

You do NOT modify:
- SemaphoreUI configuration or database
- Target server configurations
- Inventory files (SemaphoreUI manages this)

## Coding Conventions

### Playbook structure
```yaml
# ✅ Good - explicit, documented
- name: Run local script on remote hosts
  hosts: "{{ target_hosts | default('all') }}"
  gather_facts: false
  become: false

  tasks:
    - name: Run the script
      ansible.builtin.script:
        cmd: "{{ script_path | default('test.sh') }}"
        executable: /bin/bash

# ❌ Bad - missing name, no defaults
- hosts: all
  tasks:
    - script: "{{ script }}"
```

### YAML rules
- Always use explicit `ansible.builtin.` prefix for modules
- Use `| default('value')` for required variables
- Always set `gather_facts: false` unless needed (faster execution)
- Comment complex variable usage

### Script files
- Use `#!/bin/bash` or `#!/usr/bin/env bash`
- Scripts run on remote hosts, not locally
- Keep scripts self-contained (no local file dependencies)

## Testing Rules

Before any commit:
1. Run `ansible-playbook --syntax-check` on all modified playbooks
2. Test with `--check` mode against a dev server if possible
3. Validate YAML syntax: `python -c "import yaml; yaml.safe_load(open('playbooks/*.yml'))"`

Always verify:
- Variables have defaults or are required
- hosts is correctly templated
- become settings match security requirements

## Security & Sensitive Data

SSH keys:
- Never commit private keys to the repository
- Use SemaphoreUI's key management (not files in this repo)
- For local testing, use `~/.ssh/id_rsa` with proper permissions: `chmod 600 ~/.ssh/id_rsa`

Secrets handling:
- Do NOT add secrets to `ansible.cfg` or playbooks
- Use SemaphoreUI environment variables for secrets
- Never echo sensitive data in scripts

## Change Management

### Commit conventions
```
feat: add playbook for nginx deployment
fix: correct script path variable in run_script.yml
docs: update README with new example
```

### Adding new playbooks
1. Create `playbooks/new_playbook.yml`
2. Add usage comment at top
3. Run syntax check
4. Document in README.md if it's a common operation

## Boundaries & Prohibitions

- ✅ **Always**: Use `gather_facts: false` unless facts are needed, add defaults to variables, test syntax before committing
- ⚠️ **Ask first**: Before adding privilege escalation (become: true), before modifying ansible.cfg defaults
- 🚫 **Never**: Commit secrets or SSH keys, add passwords in plain text, modify SemaphoreUI configs, create inventory files (SemaphoreUI manages this)
