# Welcome to Ansible

![Ansible Logo Dark](assets/images/ansible_logo_dark.png#only-dark){: style="width:200px"}
![Ansible Logo Light](assets/images/ansible_logo_light.png#only-light){: style="width:200px"}

## What is Ansible

Ansible is a python based automation framework.

## Why use Ansible

## Ansible Initial Setup

### Config File

### Inventory

```yaml
WORKSHOP_FABRIC:
  children:
    S1:
      children:
        S1_SPINES:
          hosts:
            s1-spine1:
            s1-spine2:
        S1_LEAFS:
          hosts:
            s1-leaf1:
            s1-leaf2:
            s1-leaf3:
            s1-leaf4:
            s1-brdr1:
            s1-brdr2:
            s1-core1:
            s1-core2:
    S2:
      children:
        S2_SPINES:
          hosts:
            s2-spine1:
            s2-spine2:
        S2_LEAFS:
          hosts:
            s2-leaf1:
            s2-leaf2:
            s2-leaf3:
            s2-leaf4:
            s2-brdr1:
            s2-brdr2:
            s2-core1:
            s2-core2:
```

### Variables

```yaml title="inventory/group_vars/WORKSHOP_FABRIC.yml"

# eAPI connectivity via HTTPS (as opposed to CLI via SSH)
ansible_connection: ansible.netcommon.httpapi

# Specifies that we are using Arista EOS
ansible_network_os: arista.eos.eos

# Use SSL (HTTPS)
ansible_httpapi_use_ssl: true

# Disable SSL certificate validation
ansible_httpapi_validate_certs: false

# Credentials
ansible_user: arista
ansible_ssh_pass: [your topology password]

```

### Ad-Hoc Commands

Ping

```bash
ansible S1 -m ping -i inventory.yml
```

## Modules

## Playbooks

## Roles

## Collections
