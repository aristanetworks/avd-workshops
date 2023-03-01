# Welcome to Ansible

![Ansible Overview](assets/images/ansible_overview.png){: style="width:800px"}

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

???+ tip "Important Note Before Getting Started"

    This section will make use of the fork of the [Workshops](https://github.com/PacketAnglers/workshops "Workshops Repo on GitHub") GitHub repository that was
    created during the [Git](git.md) section. If you have not created a fork of this repository, and cloned it into the `/home/coder/project/labfiles/` directory of
    your lab environment's VS Code IDE, please do so before moving forward.

## Ansible Initial Setup

There are multiple methods of installing Ansible on the Ansible Control Node. The most popular method is to leverage `pip`, and is covered in detail [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html "Ansible Installation Documentation").

We will be using `ansible-core`, which is a lightweight minimalist installation of Ansible without any extra modules, plugins, etc. included. With this approach, we can use Ansible Galaxy (covered later in this section) to install collections containing the modules, plugins, roles, etc. that we need. For those familiar with Python, think of Ansible Galaxy as [pypi.org](https://pypi.org "Python Packge Index"), and Ansible Collections as Python modules.

??? note
    In the Arista Test Drive lab topology, Ansible is already installed, so we won't need to perform any installation related tasks.

Below is an example of installing `ansible-core` via pip on Ubuntu 20.04:

```bash
#If necessary, install Python3
sudo apt install update && upgrade
sudo apt install python3 python3-pip --yes

# Install Ansible Core
pip3 install ansible-core
```

It really is that easy to get started!

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

![Ansible Control Node](assets/images/ansible_control_node.png)

Two important initial terms are the Ansible `Control Node` and `Managed Node`

The Control Node is where our playbooks are executed from. This node then connects to the Managed Node(s) to interact with them as needed to perform the desired task(s). How the Control Node interacts with the Managed Node is dependent upon the type of operating system running on the Managed Node. For example, if the Managed Node is a Linux server, then the Control Node will "ship" the python code associated with the task(s) to the Managed Node. Then, the Managed Node will locally execute that code to complete the task(s).

??? note "What about the lab environment?"
    In our ATD lab environment, the `Control Node` is our JumpHost that we're running our VS Code IDE from.
    The `Managed Nodes` are the switches that make up our lab's network topology.

If the Managed Node is a network device, then the Control Node will locally execute the python code associated with the task(s), and then interact with the network devices via SSH or API to complete the task(s).

The Control Node must be a Linux host (Ubuntu, CentOS, Rocky, Debian, etc.) with Ansible installed. That's it! Really! This is part of what makes Ansible easy to get started with, and also efficient. It does not require a suite of software to be installed in order to get started. A single Control Node can manage hundreds, or thousands, of Managed Nodes.

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

![Ansible Config File](assets/images/ansible_config_file.png)

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

One of the most common settings in the ansible.cfg file is the location of the `inventory` file, which we will discuss next.

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

![Ansible Inventory](assets/images/ansible_inventory.png)

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

Variables can be defined in many locations with Ansible. For example, explicitly defining a variable when running a playbook by using the `extra-vars` flag

??? warning "Reminder"
    Ensure all commands are run from the `/home/coder/project/labfiles/workshops/ansible` directory in the terminal on the ATD VS Code IDE instance.

```bash
ansible-playbook playbooks/hello_world.yml -e 'name=Mitch'
```

The contents of the playbook we just ran can be seen below. We will look more into the anatomy of a playbook in our next section. For now, make note of the fact that the `name` variable is also set in the `hello-world.yml` playbook
using the `vars` parameter. When we ran our playbook with `extra_vars`, this took precedence over the variable defined inside of the playbook.

??? eos-config annotate "hello_world.yml Playbook (~/project/labfiles/workshops/ansible/playbooks/hello_world.yml)"
    ```yaml

    ---

    - name: A simple playbook
      hosts: localhost
      gather_facts: false
      vars:
        name: Mr.T

      tasks:

        - name: Say Hello
          debug:
            msg: "Hello {{ name | default('you!') }}"

    ```

Other valid locations for variables, and their respective precedence, are shown in the diagram below:

![Ansible Variables Precedence](assets/images/ansible_variables1.png)

As can be expected, there are a LOT of places where we can define variables. Each option has it's use case, but the general recommendation is to make use of ==host_vars== and ==group_vars== as much as possible.

Inside of our `~/project/labfiles/workshops/ansible/inventory` directory, there is a `host_vars` and `group_vars` directory. These are special directories that Ansible will use to establish a hierarchy of variables
that maps directly to our inventory hosts and groups structure. Each group can have a dedicated `yaml` file, as can each host. A visual representation of this can be seen below:

![Ansible Variable Visualization](assets/images/ansible_variables2.png)

In the example above, if we were to define a variable in the `~/project/labfiles/workshops/ansible/inventory/group_vars/WORKSHOP_FABRIC.yml` file, then all Managed Nodes contained within that group in our inventory file would
inherit that variable. The contents of our `WORKSHOP_FABRIC.yml` file can be seen below:

??? eos-config annotate "WORKSHOP_FABRIC.yml (~/project/labfiles/workshops/ansible/inventory/group_vars/WORKSHOP_FABRIC.yml)"
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
    ansible_ssh_pass: arista1c7z

    # Global login banner for all switches in topology
    banner_text: "This banner came from group_vars/WORKSHOP_FABRIC.YML"

    ```

These values will all be inherited by the nodes in the `WORKSHOP_FABRIC` group defined in our inventory file.

We can verify this by running `ansible-inventory --host s2-spine1 --yaml`.

```bash
ansible-inventory --host s2-spine1 --yaml
```

??? eos-config annotate "Output of 'ansible-inventory --host s1-spine1 --yaml'"
    ```yaml

    ansible_connection: ansible.netcommon.httpapi
    ansible_httpapi_use_ssl: true
    ansible_httpapi_validate_certs: false
    ansible_network_os: arista.eos.eos
    ansible_ssh_pass: arista1c7z
    ansible_user: arista
    banner_text: This banner came from group_vars/WORKSHOP_FABRIC.YML

    ```

And there they are! All of the variables we had defined in `WORKSHOP_FABRIC.yml` are present.

If we run this same command, but specifying `s1-leaf3` we'll see some additional, and slightly different, variables:

```bash
`ansible-inventory --host s1-leaf3 --yaml`
```

??? eos-config annotate "Output of 'ansible-inventory --host s1-leaf3 --yaml'"
    ```yaml

    ansible_connection: ansible.netcommon.httpapi
    ansible_httpapi_use_ssl: true
    ansible_httpapi_validate_certs: false
    ansible_network_os: arista.eos.eos
    ansible_ssh_pass: arista1c7z
    ansible_user: arista
    banner_text: This banner came from group_vars/S1.YML
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

Whoa! There is certainly more there...and looking at `banner_text`, we can see that it's different. With group_vars, the closer to the host we get, the higher the precedence of the variable. So, in the case of `banner_text`, it is defined
in `WORKSHOP_FABRIC.yml` and `S1.yml`. Because the `S1` group is closer to the host (`s1-leaf3`) in this case, the `banner_text` variable defined in `S1.yml` take priority.

??? eos-config annotate "S1.yml (~/project/labfiles/workshops/ansible/inventory/group_vars/S1.yml)"
    ```yaml

    mlag_config:
      domain_id: 1000
      vlan:
        id: 4094
        name: mlagpeer
        trunk_group_name: mlagpeer
      peer_link_id: 1000
      side_a:
        ip: 10.0.0.1/30
        peer_ip: 10.0.0.2
      side_b:
        ip: 10.0.0.2/30
        peer_ip: 10.0.0.1

    banner_text: "This banner came from group_vars/S1.YML"

    ```

We'll use the `mlag_config` stuff later. For now, let's keep our focus on the `banner_text` variable.

Finally, let's take a look at the effective variables on `s1-leaf1`

```bash
ansible-inventory --host s1-leaf1 --yaml
```

??? eos-config annotate "Output of 'ansible-inventory --host s1-leaf1 --yaml'"
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

Notice we have a few more variables now, namely the `mlag` dictionary. We also can see that the `banner_text` has changed yet again. This time due to the fact that this variable is defined in
`~/project/labfiles/workshops/ansible/inventory/host_vars/s1-leaf1.yml`. This demonstrates that a variable defined in `host_vars` will take priority over the same variable defined in `group_vars`.

??? eos-config annotate "s1-leaf1.yml (~/project/labfiles/workshops/ansible/inventory/host_vars/s1-leaf1.yml)"
    ```yaml

    mlag_config:
      domain_id: 1000
      vlan:
        id: 4094
        name: mlagpeer
        trunk_group_name: mlagpeer
      peer_link_id: 1000
      side_a:
        ip: 10.0.0.1/30
        peer_ip: 10.0.0.2
      side_b:
        ip: 10.0.0.2/30
        peer_ip: 10.0.0.1

    banner_text: "This banner came from group_vars/S1.YML"

    ```

Next, let's use a playbook to deploy our "Message of the Day" banner to all of the switches in our lab!

### Playbooks

Ansible Playbooks are written in YAML, and typically consist of four main components:

- Plays
- Tasks
- Modules
- Module Parameters

Other items such as variables, conditionals, tags, comments, and more are all available tools for us as well. But, for our discussion, we will focus on the aforementioned four main components.

To do this, we'll review a playbook together. Specifically, the **`~/project/labfiles/workshops/ansible/playbooks/deploy_banner.yml`** in our lab environment.

![Ansible Playbook Anatomy](assets/images/ansible_playbook_anatomy.png)

At the very start of our playbook, we have the ==Play==. This is the very root of the playbook. it is where we define the Managed Nodes we'd like to target with this play, as well as the list of task(s) we'd like
to run on these ==target hosts==.

Next, we have the ==task== itself, which in our case is leveraging the **eos_facts** ==module== to gather information about the Managed Nodes, which are devices running Arista's EOS, in our topology.
In a minute, we'll unpack what a module really is behind the scenes.

Finally, we have any ==parameters== associated with module. Some of these parameters may be required, while others may be optional. The best way to learn about a module is to find it's documentation out on [Ansible Galaxy](https://galaxy.ansible.com/ "Ansible Galaxy") by searching for it. For example, when we search for `eos_` on Ansible Galaxy, we get the output below:

![Ansible Galaxy Module Search Result](assets/images/ansible_module_search.png)

34 Modules, and one of which is the `eos_banner` module, with all of it's associated documentation!

More on Ansible Galaxy in a bit...

Let's go ahead and run our playbook! Bonus points if cowsay is enabled.

```bash
ansible-playbook playbooks/deploy_banner.yml
```

Alright! Let's hop into our switches and see what happened...

#### s2-spine1

```txt
"This banner came from group_vars/WORKSHOP_FABRIC.yml"
s2-spine1#
```

#### s1-spine1

```txt
"This banner came from group_vars/S1.yml"
s1-spine1#
```

#### s1-leaf1

```txt
"This banner came from host_vars/s1-leaf1.yml"
s1-leaf1#
```

As expected, each device used whichever `banner_text` variable was closest to it in the group hierarchy or, in the case of s1-leaf1, was applied via the host_vars file associated with the node.

### Modules

When we ran our playbook, one of the key components was the ==module==. In our case, this was the [eos_banner module](https://github.com/ansible-collections/arista.eos/blob/main/docs/arista.eos.eos_banner_module.rst "eos_banner Module Documentation").

But, what is this module really doing behind the scenes? Why are modules such a key piece of what makes Ansible much easier to get started with than other Automation Frameworks?

Modules are an abstraction of the Python code necessary to complete a given task, or tasks, associated with the module. In other words, behind the scenes, modules are just python code!

An example of this can be seen below, specifically for the [eos_banner module](https://github.com/ansible-collections/arista.eos/blob/main/plugins/modules/eos_banner.py "eos_banner module python code"):

![Ansible Module Anatomy](assets/images/ansible_module_anatomy.png)

It's a heck of a lot easier to simply call `eos_banner` in a playbook than it is to write, in a reusable fashion, all of that Python code! This is why modules are such as big reason Ansible is easy to get started with - the abstraction of the
detailed code behind the scenes necessary to get things done.

Now, if we feel compelled to dive in and write our own modules, or our own roles/collections/plugins, we can certainly do this. Ansible is very extensible in this manner. When we're finished developing our content, if we want to share it with the world, then we can publish it as a Collection on [Ansible Galaxy!](https://galaxy.ansible.com "Ansible Galaxy").

### Ansible Galaxy

Earlier, we specified that we were using `ansible-core` for this workshop. This approach, as opposed to installing `ansible`, is becoming more and more preferred. The reason for this is the efficiency of `ansible-core`; it is a lightweight minimalist installation of Ansible without any extra modules, roles, plugins, etc. natively included.

With `ansible-core` we can only grab the modules, roles, plugins, etc. that we need from [Ansible Galaxy!](https://galaxy.ansible.com "Ansible Galaxy").

![Ansible Galaxy](assets/images/ansible_galaxy.png){: style="width:800px"}

For example, if we want to install the [Arista EOS Ansible Collection](https://galaxy.ansible.com/arista/eos "Arista EOS Collection"), we can enter the following command in our terminal

```bash
ansible-galaxy collection install arista.eos
```

Once completed, we can validate that the collection has been installed by running the command below:

```bash
ansible-galaxy collection list
```

???+ info
    The `ansible-galaxy collection list` command is supported in ansible 2.10+

This will yield output similar to below:

??? eos-config annotate "Ansible Galaxy Collection List"
    ```powershell

    # /home/coder/.ansible/collections/ansible_collections
    Collection        Version
    ----------------- -------
    ansible.netcommon 4.1.0
    ansible.posix     1.4.0
    ansible.utils     2.8.0
    arista.avd        3.6.0
    arista.cvp        3.4.0
    arista.eos        6.0.0
    community.general 6.2.0

    ```

Note the line in the output `/home/coder/.ansible/collections/ansible_collections`...how did Ansible know to look here? The answer: The `ansible.cfg` file!

Specifically this parameter:

```yaml
# Path(s) to search for installed Ansible Galaxy Collections
collections_paths = ~/.ansible/collections
```

Galaxy has thousands of modules, plugins, roles, and more from a multitude of community members.

For those familiar with Python, we can think of Ansible Galaxy as [PyPi](https://www.pypi.org "Python Package Index"), and our Collections as Python modules.

Go forth and explore!
