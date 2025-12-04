# Ansible Project - WordPress

## Summary
Repository containing a playbook and a role (`roles/wordpress`) to deploy a basic WordPress stack (Apache + MySQL + PHP). It includes an example inventory and the main tasks required to install, configure, and start the services.

## Relevant structure
```
./
├── inventory.ini               # Host inventory
├── playbook.yml                # Main playbook (uses the 'wordpress' role)
├── roles/
│   └── wordpress/
│       └── tasks/
│           └── main.yml        # Role tasks
├── keys/                       # SSH private/public keys for access
└── README.md
```

## Requirements
- Ansible 2.9+ (2.10+ or Ansible Core 2.12+ recommended)
- SSH access to the target machines from the control node
- Python 3.x on the remote machines (for Ansible modules)
- Sudo privileges for the remote user (or use `ansible_become_password` / NOPASSWD configuration)

## Important variables
Do not store passwords in plain text. These variables are used by the playbook/role and can be set in `group_vars`, `host_vars`, passed as `--extra-vars`, or stored in an `ansible-vault` file:

- `wordpress_db_name` (default: `wordpress`)
- `wordpress_db_user` (default: `wordpress`)
- `wordpress_db_pass` (database user password)
- `wordpress_admin_user` (WordPress admin user)
- `wordpress_admin_pass` (WordPress admin password)
- `wp_install_dir` (installation path, e.g. `/var/www/html/wordpress`)

## Example `inventory.ini`
Secure example using an SSH private key:

```
[servidores_web]
ubuntu_host ansible_host=192.168.221.128 ansible_user=ubuntu ansible_ssh_private_key_file=./keys/ubuntu.pem
```

If you need to provide a sudo password for testing (not recommended for production), you can add `ansible_become_password` in the inventory. Prefer using `ansible-vault` to protect secrets:

```
ubuntu_host ansible_host=192.168.221.128 ansible_user=ubuntu ansible_ssh_private_key_file=./keys/ubuntu.pem ansible_become_password=YOUR_SUDO_PASS
```

## Useful commands

- Test connectivity to hosts:
```bash
ansible -i inventory.ini -m ping servidores_web
```

- Run the playbook and prompt for the sudo password:
```bash
ansible-playbook -i inventory.ini playbook.yml -K
```

- Run the playbook using `ansible-vault` (recommended for secrets):
```bash
# Create an encrypted variables file
ansible-vault create vault.yml

# Run using the vault file
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass -K
```

- Run with inline variables (temporary):
```bash
ansible-playbook -i inventory.ini playbook.yml -K --extra-vars "wordpress_db_pass='MyPass' wordpress_admin_pass='AdminPass'"
```

## Security and best practices
- Do not store passwords in plain text in inventory or playbooks. Use `ansible-vault` to encrypt secrets.
- For CI/CD automation, prefer `--vault-password-file` stored securely or integrate with your runner's secret store.
- In controlled environments you may configure passwordless sudo for the Ansible user (`/etc/sudoers.d/`) but restrict allowed commands rather than using `NOPASSWD:ALL`.

## Notes and troubleshooting
- If Ansible reports a missing sudo password, re-run with `-K` or provide `ansible_become_password`/`ansible-vault`.
- If the role relies on `wp` (WP-CLI) and it is not available, the role attempts to install `wp-cli` automatically; on distributions where the package is unavailable you may need to install PHP and additional dependencies or download the WP-CLI phar.

If you want, I can:
- Add an example `group_vars/servidores_web.yml` with default variables.
- Create an encrypted example using `ansible-vault` and show how to use it.
- Adjust the playbook/role to install WP-CLI from the phar when the distribution package is not available.

---
Date: 2025-12-03
