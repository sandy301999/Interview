Ansible Topics
================================================
Writing playbooks
Using roles for modularization
Defining and managing variables
Encrypting sensitive data with Ansible Vault
Ensuring idempotent tasks
Configuration management (e.g., OS hardening)
File transfers and updates
Script deployment
Managing cron jobs
Using handlers for service restarts and notifications
Gathering and leveraging facts
Using Ansible Vault for secrets management
Leveraging blocks, include, and import statements
Using debug for troubleshooting
Managing group variables (group_vars)
Organizing and managing inventories
Splitting configurations into modular files
Polling and rolling deployments
Building reusable roles

ansible-infra/
├─ inventories/
│  ├─ dev/
│  │  └─ inventory.ini
│  └─ prod/
│     └─ inventory.ini
├─ group_vars/
│  ├─ all.yml
│  ├─ web.yml
│  └─ db.yml
├─ roles/
│  ├─ common/
│  │  ├─ tasks/main.yml
│  │  ├─ handlers/main.yml
│  │  ├─ vars/main.yml
│  │  ├─ defaults/main.yml
│  │  └─ templates/
│  └─ webserver/
│     ├─ tasks/main.yml
│     ├─ handlers/main.yml
│     └─ templates/nginx.conf.j2
├─ playbooks/
│  ├─ site.yml
│  ├─ users.yml
│  └─ security.yml
├─ vault/
│  └─ secrets.yml   (encrypted)
└─ README.txt
