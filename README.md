## Z10 Ansible

Ansible configuration for the z10 fedora server.

## Inventory

- Z10 - Fedora server

#Current configuration

- Git
- cur
- vim

## bash
```bash
ansible-playbook -i inventory.ini site.yml --ask-become-pass
```
