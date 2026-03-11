# Ansible Setup for SemaphoreUI

Minimal Ansible configuration for running remote scripts via SemaphoreUI.

## Structure

```
.
├── ansible.cfg          # Ansible configuration
├── playbooks/
│   ├── run_script.yml   # Playbook to run scripts on remote hosts
│   └── test.sh          # Example script to run
└── README.md
```

## SemaphoreUI Setup

1. **Add Environment**: In SemaphoreUI, create an environment with:
   - SSH key (handled by SemaphoreUI)
   - No extra variables needed

2. **Add Inventory**: Create inventory in SemaphoreUI with your target hosts

3. **Create Key**: Add SSH key in SemaphoreUI for authentication

4. **Create Template**: Create a template using:
   - **Playbook**: `playbooks/run_script.yml`
   - **Inventory**: Your inventory
   - **Environment**: Your environment

## Running Scripts

### Via SemaphoreUI
1. Create a new template in SemaphoreUI
2. Use `playbooks/run_script.yml` as the playbook
3. Add extra vars: `script_path=playbooks/test.sh`
4. Run the template

### Via CLI (for testing)
```bash
ansible-playbook playbooks/run_script.yml \
  -i "server1.example.com," \
  -e "script_path=playbooks/test.sh" \
  --user your_ssh_user \
  --private-key ~/.ssh/id_rsa
```

## Customizing

Edit `playbooks/run_script.yml` to change behavior:
- Remove `become: false` to run as root
- Add more tasks as needed
