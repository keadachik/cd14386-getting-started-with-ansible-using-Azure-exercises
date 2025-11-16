## How to Run

Prerequisites:

```bash
ansible-galaxy collection install ansible.posix
# If you prefer YAML stdout formatting (optional):
# ansible-galaxy collection install community.general
```

Connectivity check:

```bash
cd project
ansible -i inventory.ini azure -m ping
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml
```


## Evidence (Recommended)

- Include a short log excerpt showing `failed=0` for all 3 hosts.
- On the second run, the directory check should print:
  - `/var/www/html/www.companyplus.com exists.`


## Version Notes (Optional)

```bash
ansible --version
```

- Note: Some environments may show a warning about `ansible.posix` not officially supporting Ansible 2.14.x. In most cases it still works; the warning can be ignored or resolved by upgrading ansible-core.


## Optional Enhancements

- Place an `index.html` under `document_root/app_root` (default: `/var/www/html/html_demo_site-main`) and verify HTTP access:

```bash
# On one of the target hosts
echo "hello" | sudo tee /var/www/html/html_demo_site-main/index.html

# From your machine
curl http://<VM_IP>/
```

- Add brief notes about any extra modules, conditionals, or filters you used.


## Structure Overview

- `inventory.ini`: defines the `azure` group with 3 hosts.
- `group_vars/azure/vars.yml`: defines `ports: [80, 443]` (applied to firewalld).
- `playbook.yml`:
  - installs/enables firewalld
  - opens ports via `ansible.posix.firewalld` looping over `ports` (permanent + immediate)
  - checks for `/var/www/html/www.companyplus.com` and prints a conditional message
  - calls the `webserver` role with:
    - `app_root: html_demo_site-main`
    - `server_name: "{{ ansible_default_ipv4.address }}"`
    - `document_root: /var/www/html`
- `roles/webserver`:
  - `vars/main.yml`: `webserver: nginx`
  - `tasks/main.yml`: installs nginx, creates 2 content directories (owner/group=nginx, mode=0770), deploys template
  - `handlers/main.yml`: restarts nginx after the template is updated
  - `templates/nginx.conf.j2`: uses `app_root`, `server_name`, `document_root`


