# Welcome to Ansible

![Ansible Overview](assets/images/ansible_overview.png)

???+ tip "Important Note Before Getting Started"

    This section will make use of the fork of the [Workshops](https://github.com/PacketAnglers/workshops "Workshops Repo on GitHub") GitHub repository that was
    created during the [Git](git.md) section. If you have not created a fork of this repository, and cloned it into your lab environment's VS Code IDE, please do so
    before moving forward.

## What is Ansible

Ansible is a python based automation framework. Today, the term "Ansible Automation Platform" can refer to multiple applications, including:

- Ansible Core
- Ansible Galaxy
- Ansible Automation Controller (Previously known as Tower)
- Red Hat Insights

We will be focusing on the first two items during this workshop.

## Why use Ansible

Aside from being 'agentless', meaning that Ansible does not require any specialized software on the target hosts, Ansible is also easy to get started with. While prior coding or experience in automation is of course helpful, it is not required to get up and running with Ansible. Playbooks are written in YAML, a language that we'll cover in detail in the [YAML](jinja-yaml.md "Jinja and YAML") section. For now, rest assured that YAML is a human readable language, and it's a big reason why it's easy to get started on our Ansible journey.

There is a very large, and very active, user and development community with Ansible. The project itself is open source, with the GitHub repository available [here](https://github.com/ansible/ansible "Ansible on GitHub"). The popularity of Ansible has led to broad vendor support, spanning multiple technology silos. Network, Compute, Storage, Cloud, Security, and more can all be automated via Ansible.

Finally, all that is required to get started is a Linux host with Python installed. A single Ansible Control Node (ACN) can manage hundreds, or thousands, of endpoints.

## Ansible Initial Setup

There are multiple methods of installing Ansible on the Ansible Control Node. The most popular method is to leverage `pip`, and is covered in detail [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html "Ansible Installation Documentation").

We will be installing `ansible-core`, which is a lightweight minimalist installation of Ansible without any extra modules, plugins, etc. included. With this approach, we can use Ansible Galaxy (covered later in this section) to install collections containing the modules, plugins, roles, etc. that we need. For those familiar with Python, think of Ansible Galaxy as [pypi.org](https://pypi.org "Python Packge Index"), and Ansible Collections as Python modules.

Below is an example of installing `ansible-core` via pip on Ubuntu 20.04:

```bash
#If necessary, install Python3
sudo apt install update && upgrade
sudo apt install python3 python3-pip --yes

# Install Ansible Core
pip3 install ansible-core
```

It really is that easy to get started!

In the Arista Test Drive lab topology, Ansible is already installed, so we won't need to perform any installation related tasks.

Before running any commands, let's first ensure that we're in the `/home/coder/project/labfiles/workshops/ansible` directory

```bash
cd ~/project/labfiles/workshops/ansible
```

Next, we'll confirm that Ansible is installed by running the `ansible --version` in the terminal. This should yield output similar to below:

```powershell
ansible [core 2.12.10]
  config file = /home/coder/project/labfiles/workshops/ansible/ansible.cfg
  configured module search path = ['/home/coder/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/coder/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/coder/.ansible/collections
  executable location = /home/coder/.local/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True
```

All of the information displayed above is important, and can help when troubleshooting why something may not be working as expected when working with Ansible.

## Control Node and Managed Nodes

![Ansible Overview](assets/images/ansible_control_node.png)

Two important initial terms are the Ansible `Control Node` and `Managed Node`

The Control Node is where our playbooks are executed from. This node then connects to the Managed Node(s) to interact with them as needed to perform the desired task(s). How the Control Node interacts with the Managed Node is dependent upon the type of operating system running on the Managed Node. For example, if the Managed Node is a Linux server, then the Control Node will "ship" the python code associated with the task(s) to the Managed Node. Then, the Managed Node will locally excute that code to complete the task(s).

??? note "What about the lab environment?"
    In our ATD lab enviornment, the `Control Node` is our JumpHost that we're running our VS Code IDE from.
    The `Managed Nodes` are the switches that make up our lab's network topology.

If the Managed Node is a network device, then the Control Node will locally execute the python code assocaited with the task(s), and then interact with the network devices via SSH or API to complete the task(s).

The Control Node must be a Linux host (Ubuntu, CentOS, Rocky, Debian, etc.) with Ansible installed. That's it! Really! This is part of what makes Ansible easy to get started with, and also efficient. It does not require a suite of software to be installed in order to get started. A single Control Node can manage hudreds, or thousands, of Managed Nodes.

A Managed Node does not need any specialized software installed. In other words, Ansible is `agentless`. If a Managed Node is a Linux server, then it will need to have Python3 installed. However, for Managed Nodes that are network devices, there are no pre-requisites required; Not even Python. This is because the Control Node will locally execute the python code necessary to complete the task(s) on the network device, and will then interact as needed with the network device via SSH/API.

## Ansible Components

Ansible, just like any other framework, is made up of a group of components that come together to make the magic happen. The components that we'll be focusing on during this workshop are shown below.

![Ansible Overview](assets/images/ansible_components.png)

In the next sections, we'll go through each of these components one at a time to get a better understanding of how each piece fits together to ultimately make the magic happen.

### Config File

The Ansible configuration file is where we set environment variables for our Ansible project(s). There are many variables that can be set in this file, and the most common ones are documented [here](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#common-options "Common Ansible Configuration File Options").

When running an ad-hoc command, or playbook, Ansible will look for the configuration file in the locations listed below. These locations are defined in order of precedence:

1. ANSIBLE_CONFIG (environment variable if set)

2. ansible.cfg (in the current directory)

3. ~/.ansible.cfg (in the home directory)

4. /etc/ansible/ansible.cfg

This is illustrated in the image below:

![Ansible Overview](assets/images/ansible_config_file.png)

Once Ansible finds an `ansible.cfg` file, it will only use the configuration options defined in that file. If, for example, an `ansible.cfg` file exists in the current directory, and in `/etc/ansible/ansible.cfg`, then only the settings found in the `ansible.cfg` file in the current directory will be used.

??? question "Cows?"

    Yes! First, install cowsay: `sudo apt-get update && sudo apt-get install cowsay -y`
    Next, in the `ansible.cfg` file, set it to what should be the only acceptable setting `nocows = False`.
    This feature is udderly ridiculous, and as much as I'd love to milk it for all the Dad jokes possible,
    I don't want to start any beef by delaying the workshop...Moo.

Below is an example of the `ansible.cfg` located in our fork of the [Workshops](https://github.com/PacketAnglers/workshops "Workshops Repo on GitHub") repo:

??? eos-config annotate "Example Ansible Configuration File (~/project/labfiles/workshops/ansible/ansible.cfg)"
    ```apache

    [defaults]

    # Disable host key checking by the underlying tools Ansible uses to connect to target hosts
    host_key_checking = False

    # Location of inventory file containing target hosts
    inventory = ./inventory/inventory.yml

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

    # Disable cowsay (Why?)
    nocows = True

    [paramiko_connection]
    # Automatically add the keys of target hosts to known hosts
    host_key_auto_add = True

    [persistent_connection]
    # Set the amount of time, in seconds, to wait for response from remote device before timing out persistent connection.
    command_timeout = 60

    # Set the amount of time, in seconds, that a persistent connection will remain idle before it is destroyed.
    connect_timeout = 60

    ```

One of the most common setings in the ansible.cfg file is the location of the `inventory` file, which we will discuss next.

### Inventory

The inventory file is where we define our hosts, groups that we'll be targeting with our playbooks. The inventory file supports many different formats, but the most common are `ini` and `yaml`. For our workshop, we'll be using the `yaml` format.

An example of our inventory file can be seen below:

??? eos-config annotate "Example Inventory File (~project/labfiles/workshops/ansible/inventory/inventory.yml)"
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

This inventory file defines the hosts, and groupings, represented in the image below:

![Ansible Overview](assets/images/ansible_inventory.png)

We can validate our inventory by using the `ansible-inventory` command, which is documented [here](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html#verifying-the-inventory "Verifying Ansible Inventory")

The below command will list the entire inventory, consisting of all hosts/groups and their respective variable values.

```bash
ansible-inventory --list --yaml
```

If we'd like to get more specific, we can filter the output down to a single host by using the command shown below:

```bash
ansible-inventory --host s1-leaf1 --yaml
```

??? eos-config annotate "Example inventory output for s1-leaf1"
    ```yaml
    ansible_connection: ansible.netcommon.httpapi
    ansible_httpapi_use_ssl: true
    ansible_httpapi_validate_certs: false
    ansible_network_os: arista.eos.eos
    ansible_ssh_pass: arista1c7z
    ansible_user: arista
    banner_text: This banner came from host_vars/s1-leaf1.YML
    mlag:
      enabled: true
      peer_link_int_1: 1
      peer_link_int_2: 6
      side_a: true
    mlag_config:
      domain_id: 1000
      peer_link_id: 1000
      side_a:
        ip: 10.0.0.1/30
        peer_ip: 10.0.0.2
      side_b:
        ip: 10.0.0.2/30
        peer_ip: 10.0.0.1
      vlan:
        id: 4094
        name: mlagpeer
        trunk_group_name: mlagpeer

    ```

Notice the variables associated with `s1-leaf1`. Where did these come from? We'll be exploring that next...

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
