# Ansible: Configure Azure NGINX Server

This README explains how to run the Ansible playbook using the role structure that we outlined:

```
ansible/
├── inventory.ini
├── site.yml
└── roles
    └── nginx
        ├── tasks
        │   └── main.yml
        └── templates
            └── index.html.j2
```
Terraform created VM f.e. (Ubuntu, reachable via SSH) to configure it with Ansible so that NGINX is installed, enabled, and serving a custom “hello” page.

## Prerequisites

Terraform VM is up
You should already have run
```
terraform apply
```
and have the VM’s public IP.
You can SSH into the VM manually
```
ssh azureuser@<PUBLIC_IP> -i ~/.ssh/id_rsa
```
azureuser must match what you used in Terraform.
use the right key file.
Check if Ansible is installed on your local machine
```
ansible --version
```
Hint:
```
pip install ansible
```
## Directory Structure
```
ansible/
├── inventory.ini
├── site.yml
└── roles
    └── nginx
        ├── tasks
        │   └── main.yml
        └── templates
            └── index.html.j2
```
## Files

inventory.ini - Host, User, Path to private SSH key.
More servers can be added to [nginx_servers]
site.yml - entry point playbook 
roles/nginx/tasks/main.yml - machine configuration (idempotent)
roles/nginx/templates/index.html.j2 - Jinja2 template for the HTML that will be served

## How to run
```
ansible-playbook -i inventory.ini site.yml
```
Ansible Steps:  
Ansible reads inventory.ini, finds azure-nginx and its SSH info  
connects over SSH  
runs the site.yml play  
site.yml calls the nginx role  
the role installs nginx, drops the file, starts the service 

Check in Browser:  
http://(use your VM IP) 

## Common Issues
<u>Permission denied (publickey)</u>  
check ansible_ssh_private_key_file path  
check file permissions (chmod 600 ~/.ssh/id_rsa)  
check username (azureuser vs ubuntu)  

<u>Ubuntu asks for become password</u>  
We used passwordless sudo in Terraform images (default Ubuntu cloud images usually allow it for the created user). If your image doesn’t, add this to inventory.ini 
```
ansible_become_password=YOURPASSWORD
```
Use: 
```
export YOURPASSWOR=typeyourpasswordhere
```
NEVER USE CLEARTYPE PASSWORDS IN REPOSITORIES!!!  
or  
```
ansible-playbook -i inventory.ini site.yml --ask-become-pass
```

## Roles 
cleaner playbooks: site.yml stays tiny  
reusable: you can use the same nginx role for future VMs  
extensible: add defaults/ and vars/ later for tuning  
standard Ansible layout: other people know where to look 

## Optional Steps
add group_vars/nginx_servers.yml to set variables per group  
add TLS tasks to the role  
add a handler to restart nginx only when template changes  
call this Ansible playbook from your CI (GitHub Actions) after Terraform 

## Quick Checklist
 Terraform VM is reachable by SSH  
 inventory.ini has correct IP, user, key  
 ansible-playbook -i inventory.ini site.yml runs without errors  
 Browser shows your HTML  


