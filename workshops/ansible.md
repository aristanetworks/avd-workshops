# Welcome to Ansible

![Ansible Logo Dark](assets/images/ansible_logo_dark.png#only-dark){: style="width:200px"}
![Ansible Logo Light](assets/images/ansible_logo_light.png#only-light){: style="width:200px"}

## What is Ansible

Ansible is a python based automation framework. Today, the term "Ansible Automation Platform" refers to multiple applications, including:

- Ansible Core
- Ansible Galaxy
- Ansible Automation Controller (Previously known as Tower)
- Red Hat Insights

We will be focusing on the first two items during this workshop.

## Why use Ansible

Aside from being 'agentless', meaning that Ansible does not require any specialized software on the target hosts, Ansible is also easy to get started with. While prior coding or experience in automation is of course helpful, it is not required to get up and running with Ansible. Playbooks are written in YAML, a language that we'll cover in detail in the [YAML](jinja-yaml.md "Jinja and YAML") section. For now, rest assured that YAML is a human readable language, and it's a big reason that getting started with Ansible does not require an extensive background in development.

There is a very large, and very active, user and development community with Ansible. The project itself is open source, with the GitHub repository available [here](https://github.com/ansible/ansible "Ansible on GitHub"). The popularity of Ansible has led to broad vendor support, spanning multiple technology silos. Network, Compute, Storage, Cloud, Security, and more can all be automated via Ansible.

Finally, all that is required to get started is a Linux host with Python installed. A single Ansible Control Node (ACN) can manage hundreds, or thousands, of endpoints.

## Ansible Initial Setup

There are multiple methods of installing Ansible on the Ansible Control Node. The most popular method is to leverage `pip`, and is covered in detail [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html "Ansible Installation Documentation").

We will be installing `ansible-core`, which is a lightweight minimalist installation of Ansible without any extra modules, plugins, etc. included. With this approach, we can use Ansible Galaxy (covered later in this section) to install collections containing only the modules, plugins, roles, etc. that we need. For those familiar with Python, think of Ansible Galaxy as [pypi.org](https://pypi.org "Python Packge Index"), and Ansible Collections as Python modules.

Below is an example of installing `ansible-core` via pip on Ubuntu 20.04:

```bash
#If necessary, install Python3
sudo apt install update && upgrade
sudo apt install python3 python3-pip --yes

# Install Ansible Core
pip3 install ansible-core
```

It really is that easy to get started!

In the Arista Test Drive lab topology, Ansible is already installed, so we won't need to perform any installation related tasks. However, we should still validate that Ansible is installed.

Running `ansible --version` in the terminal of your lab instance of VS Code should yield output similar to below:

```powershell
ansible [core 2.12.10]
  config file = /home/coder/.ansible.cfg
  configured module search path = ['/home/coder/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/coder/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/coder/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/coder/.local/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True
```

All of the information displayed above is important, and can help when troubleshooting why something may not be working as expected when working with Ansible. For now, we are going to focus on the Ansible configuration file, which defined above via the `config file` parameter.

### Config File

The Ansible configuration file is where we set environment variables for our Ansible project(s). There are many variables that can be set in this file, and the most common ones are documented [here](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#common-options "Common Ansible Configuration File Options").

Precedence

1. ANSIBLE_CONFIG (environment variable if set)

2. ansible.cfg (in the current directory)

3. ~/.ansible.cfg (in the home directory)

4. /etc/ansible/ansible.cfg

??? eos-config annotate "Example Ansible Configuration File (~/projects/ansible.cfg)"
    ```apache

    [defaults]

    # Disable host key checking by the underlying tools Ansible uses to connect to target hosts
    host_key_checking = False

    # Location of inventory file containing target hosts
    inventory = ~/projects/inventory/inventory.yml

    # Only gather Ansible facts if explicity directed to in a given play
    gathering = explicit

    # Disable the creation of .retry files if a playbook fails
    retry_files_enabled = False

    # Path(s) to search for installed Ansible Galaxy Collections
    collections_paths = ~/.ansible/collections

    # Enable additional Jinja2 Extensions (https://jinja.palletsprojects.com/en/3.1.x/extensions/)
    jinja2_extensions =  jinja2.ext.loopcontrols,jinja2.ext.do,jinja2.ext.i18n

    # Enable the YAML callback plugin, providing much easier to read terminal output. (https://docs.ansible.com/ansible/latest/plugins/callback.html#callback-plugins)
    stdout_callback = yaml

    # Permit the use of callback plugins when running ad-hoc commands
    bin_ansible_callbacks = True

    # List of enabled callbacks. Many callbacks shipped with Ansible are not enabled by default
    callback_whitelist = profile_roles, profile_tasks, timer

    # Maximum number of forks that Ansible will use to execute tasks on target hosts
    forks = 15

    [paramiko_connection]
    # Automatically add the keys of target hosts to known hosts
    host_key_auto_add = True

    [persistent_connection]
    # Set the amount of time, in seconds, to wait for response from remote device before timing out persistent connection.
    command_timeout = 60

    # Set the amount of time, in seconds, that a persistent connection will remain idle before it is destroyed.
    connect_timeout = 60

    ```

### Inventory

??? eos-config annotate "Example Inventory File in YAML Format (~/projects/inventory/inventory.yml)"
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

??? eos-config annotate "Example group_vars File (~/projects/inventory/group_vars/WORKSHOP_FABRIC.yml)"
    ```yaml

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
