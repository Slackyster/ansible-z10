#Z10 Ansible

Ansible configuration for the z10 fedora server.

## Inventory

- Z10 - Fedora server

#Current configuration

- Git

##

bash
ansible-playbook -i inventory.ini site.yml --ask-become-pass
