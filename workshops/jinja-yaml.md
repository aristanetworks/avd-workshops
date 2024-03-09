# Welcome to YAML & Jinja

![Jinja YAML Banner](assets/images/jinja-yaml-banner.png)

This section will cover both Jinja and YAML, which are two interdependent pieces of basic configuration automation framework. While both YAML and Jinja can get relatively complex with what they can accomplish and what can be done with them, we will only cover what is necessary to utilize these tools for network automation and DevOps. At the end of the YAML and Jinja sections, we will tie everything together with a final output.

## What is YAML?

YAML is what's officially referred to as a data serialization language. While that sounds complex, data serialization is the process of converting objects within a data model into a byte stream for the purpose of storing or transferring it. Breaking this down, we know when trying to automate configuration management or deployment, we need to specify ***what*** it is we want to configure, that is, what should our desired end state configuration look like. ( **intent based networking** )

An important note is about why we are talking about and using YAML in the first place. As mentioned in the beginning, YAML is a data serialization language, however, it is not the only one. Some other common, data serialization languages are XML, JSON, and CSV. The reason we are particular towards YAML is that not only are there libraries available in most programming languages, but also because as the following table shows, it is very human readable.

<table>
<tr>
<th>XML</th>
<th>JSON</th>
<th>YAML</th>
</tr>
<tr>
<td>

```xml
<ntp>
  <local_interface>
    <name>Management1</name>
    <vrf>mgmt</vrf>
  </local_interface>
  <servers>
    <name>time-a-g.nist.gov</name>
    <preferred>true</preferred>
    <vrf>mgmt</vrf>
  </servers>
  <servers>
    <name>utcnist.colorado.edu</name>
    <vrf>mgmt</vrf>
  </servers>
</ntp>
```

</td>
<td>

```json
{
  "ntp": {
    "local_interface": {
      "name": "Management1",
      "vrf": "mgmt"
    },
    "servers": [
      {
        "name": "time-a-g.nist.gov",
        "preferred": true,
        "vrf": "mgmt"
      },
      {
        "name": "utcnist.colorado.edu",
        "vrf": "mgmt"
      }
    ]
  }
}
```

</td>
<td>

```yaml
---
ntp:
  local_interface:
    name: Management1
    vrf: mgmt
  servers:
    - name: time-a-g.nist.gov
      preferred: true
      vrf: mgmt
    - name: utcnist.colorado.edu
      vrf: mgmt
```

</td>
</tr>
</table>

## Common YAML Syntax

Now that we know the idea of what YAML is and what it's used for, we can cover the various constructs within a YAML file, and how to use those expressions to create files that are useful to us as engineers trying to manage configurations.

### Comments

Comments are an important part of any documentation or configuration you are doing. When creating or modifying any network configurations, while you don't realize it, you are commenting something that is important. When configuring an interface, you may add a description, such as the name of a downstream device or server it's connected to. Or if it's a provider handoff you may add the NOC telephone number and the circuit ID. If you are configuring BGP peers, you may add a description to each peer so it's clear what the neighbor is.

In YAML, you can enter comments anywhere you'd like, and ideally, where it makes sense to help explain what your data model is representing. Comments in YAML are represented with the pound symbol (as shown below).

```yaml
---
# This is my YAML vars file for global configuration settings.
some config:  here

# Management Interface Configuration
mgmt_gateway: 192.168.0.1
mgmt_interface: Management0
mgmt_interface_vrf: default
```

???+ tip
    :writing_hand: In VS Code, you can auto-comment any text you want by selecting the text and pressing ***windows*** ++ctrl++ + ++slash++ or ***mac*** ++cmd++ + ++slash++.

### Mappings

The first, and basic YAML construct is a mapping, which comprises of what is technically called a `scalar`. This will probably be the only time you hear or reference the word **scalar**, ever.

```yaml
---
# These are some examples of scalars

integer: 10
boolean: true
string: "Welcome to the Automation Workshop"
```

Sometimes, a scalar is referred to as only the initial part of the line of text such as a list of names or locations (as shown below).

```yaml
---
# Another scalar example

- White Rim Trail
- Hells Revenge
- Long Canyon Road
- Black Bears Pass
- Ophir Pass
```

In normal day-to-day use, you will see scalars referred to as **key-value pair mappings**, or just **mappings** for short. A mapping is the data label (key), followed by its value, separated by a **`:`** colon.

Here are some network config specific mappings:

```yaml
---
# Network Config Mappings

mgmt_interface: Management1
mgmt_interface_vrf: MGMT

spanning_tree_mode: rapid-pvst
spanning_tree_priority: 4096
```

### Boolean

Another type of mapping in YAML are booleans. Booleans are simply a `true` or `false` value assigned to its data label. While the actual true or false does not have to be in a certain case, if you want the value to be compatible with lint options, you should use all lowercase.

Here are some network config boolean mappings.

```yaml
---
# Network Config Mappings

evpn_import_pruning: true

evpn_gateway:
        evpn_l2:
          enabled: true

mlag: false
```

### Lists

Lists, which are sometimes also called sequences, are similar to arrays in development. A list has a top level label, or what I like to refer to as a key, which represent the elements in the list. ***( The key nomenclature will come in handy when thinking through loops in Jinja templates )***  In a standard list, you will have singular entries below the top level label.

Here are some examples of lists:

```yaml
---
# Some lists of BGP specific configurations

bgp-peers:
    - 192.168.103.1
    - 192.168.103.3
    - 192.168.103.5

bgp_defaults:
    - 'no bgp default ipv4-unicast'
    - 'distance bgp 20 200 200'
    - 'neighbor default send-community'
    - 'graceful-restart restart-time 300'
    - 'graceful-restart'
```

### Dictionary

While we just covered lists, which allow you to specify an item that falls inline below a parent label or key, what happens when you want to include more specific attributes to that parent label, or if you want to introduce more details such as key-value pair mappings to your list? In that occasion, what is used, and what you will see regularly in just about every network vars file, is a dictionary. As previously described, a dictionary is a list of key-value mappings.

Here is an example of a dictionary:

```yaml
---
l3spine:
  defaults:
    platform: 720XP
    spanning_tree_mode: rapid-pvst
    spanning_tree_priority: 4096
    loopback_ipv4_pool: 10.0.0.0/24
    mlag_peer_ipv4_pool: 192.0.0.0/24
    mlag_peer_l3_vlan: false
    virtual_router_mac_address: aa:aa:bb:bb:cc:cc
    mlag_interfaces: [Ethernet53, Ethernet54]
    mlag_port_channel_id: 2000
```

### Nested Data Structures

One of the double-edged parts of YAML is that it can be quite complex. This is good because it is very flexible for how we can build our data model, but it also means that model can get out of control. YAML syntax is hierarchical, and at the same time, just about every construct we covered previous to this can be nested within each other. Let's take a look at some nested data structures that illustrate this.

In this first example we will have a list of dictionaries:

???+ tip "Reminder"
    :writing_hand: Indentation is key.

```yaml
---
# local users

local_users:
  admin:
    privilege: 15
    role: network-admin
    sha512_password: encrypted_pass
  cvpadmin:
    privilege: 15
    role: network-admin
    sha512_password: encrypted_pass
  ansible_user:
    privilege: 15
    role: network-admin
    sha512_password: encrypted_pass
  noc:
    privilege: 1
    role: network-operator
    sha512_password: encrypted_pass
```

The following example will show another network device specific nested data structure which mixes dictionaries and lists in different places:

```yaml
---
ntp:
  local_interface:
    name: Management1
    vrf: MGMT
  servers:
    - name: 0.north-america.pool.ntp.org
      preferred: true
      vrf: MGMT
    - name: 1.north-america.pool.ntp.org
      vrf: MGMT
    - name: 2.north-america.pool.ntp.org
      vrf: MGMT

svis:
  10:
    name: 'DATA'
    tags: ['Web']
    enabled: true
    ip_virtual_router_addresses:
      - 10.10.10.1
    mtu: 9214
    ip_helpers:
      10.100.100.20:
        source_interface: Management1
        source_vrf: MGMT
      127.0.0.1:
    nodes:
      SW-CORE-A:
        ip_address: 10.10.10.2/23
      SW-CORE-B:
        ip_address: 10.10.10.3/23
```

## YAML File Examples

Now that we have gone through all the most common constructs within a YAML file used for variables for network device configs, let's see what some complete files would look like for a real world environment.

### Network Config Example

These example YAML files could be used to build a base config for a series of devices. The devices would be based on your inventory file, however, for this example, we will assume there is a layer2 leaf/spine topology, with two spines and two leafs. For this example we will use two files, a `global.yml` and a `interface.yml` file.

The global.yml file that follows includes the data model used for the base configuration. These items apply to all four devices in the fabric. ***( This could be imported in the playbook, or put in the `group_vars` directory and named after the level of your hierarchy that contains the devices you want this to apply to. )***

`global.yml`

```yaml
---
# local users
local_users:
  admin:
    privilege: 15
    role: network-admin
    secret: aristaadmin
  noc:
    privilege: 1
    role: network-operator
    secret: aristaops

# aaa authentication and authorization
aaa_authentication:
  login:
    default: group radius local

aaa_authorization:
  exec:
    default: group radius local

# radius servers
radius_servers:
  - host: 192.168.1.10
    vrf: MGMT
    key: 055A0A5D311E1E5B4944

# HTTP Client source interface and VRF
ip_http_client_source_interfaces:
    - name: Management1
      vrf: MGMT

# RADIUS source interface and VRF
ip_radius_source_interfaces:
  - name: Management1
    vrf: MGMT

#MAC and ARP aging timers
mac_address_table:
  aging_time: 1800

arp:
  aging:
    timeout_default: 1500

# DNS Servers
name_servers:
  - 10.100.100.20

# DNS lookup source interface (Servers defined in 1L2P.yml)
ip_domain_lookup:
  source_interfaces:
    Management1:
      vrf: MGMT

# NTP Servers (source interface defined in group specific YML files (CORE, ACCESS, MGMT, INET)
ntp:
  local_interface:
    name: Management1
    vrf: MGMT
  servers:
    - name: 0.north-america.pool.ntp.org
      preferred: true
      vrf: MGMT
    - name: 1.north-america.pool.ntp.org
      vrf: MGMT
    - name: 2.north-america.pool.ntp.org
      vrf: MGMT
    - name: time.google.com
      vrf: MGMT

clock:
  timezone: "America/Detroit"
```

`interface.yml`

```yaml
---
spine1:
  mlag_side: A
  interfaces:
    Ethernet47:
      desc: To_SPINE2_MLAG_PEERLINK
      mlag_peerlink: true
    Ethernet48:
      desc: To_SPINE2_MLAG_PEERLINK
      mlag_peerlink: true
    Ethernet1:
      desc: TO_LEAF1
      mlag_peerlink: false
    Ethernet2:
      desc: TO_LEAF2
      mlag_peerlink: false
spine2:
  mlag_side: B
  interfaces:
    Ethernet47:
      desc: 'To_SPINE1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet48:
      desc: 'To_SPINE1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet1:
      desc: 'TO_LEAF1'
      mlag_peerlink: false
    Ethernet2:
      desc: 'TO_LEAF2'
      mlag_peerlink: false
leaf1:
  mlag_side: A
  interfaces:
    Ethernet47:
      desc: 'To_LEAF2_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet48:
      desc: 'To_LEAF2_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet1:
      desc: 'TO_SPINE1'
      mlag_peerlink: false
    Ethernet2:
      desc: 'TO_SPINE2'
      mlag_peerlink: false
leaf2:
  mlag_side: B
  interfaces:
    Ethernet47:
      desc: 'To_LEAF1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet48:
      desc: 'To_LEAF1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet1:
      desc: 'TO_SPINE1'
      mlag_peerlink: false
    Ethernet2:
      desc: 'TO_SPINE2'
      mlag_peerlink: false
```

# Jinja

## What is Jinja?

At a high level, Jinja is a templating engine used to create markup files such as HTML or XML as well as custom text files, such as in our instance config files. Under the hood, Jinja is an open source python library that let's you create extensible templates. One of the major benefits of Jinja is that the template files you create allow you to define both static text, as well as variables. Some of the Jinja template syntax may look familiar because Jinja is not the first templating engine, and is actually inspired by Django.

## What is Jinja Used For?

As it may have become apparent from the YAML section, after creating our data model of the various configuration parameters we want to automate, we need to get that data model and all its variables into a format that can be read and understood by our network devices. This is where Jinja comes in. The use of Jinja templates, along with some yet to be shown Ansible magic, allows us to render full or partial configuration files that can loaded onto network devices. The underlying purpose of this, as it relates to automation, is that with the use of various expressions and variables in the Jinja templates, we can use a single template, with single or multiple YAML variable files, and create configurations against a multitude of network devices. As far as the actual template file and its file extension, technically any file can be called as a template regardless of its extension as long as its formatted correctly, however, we typically use the `.j2` extension on all Jinja template files.

## Jinja Syntax

As a foreword to getting into the different tasks we can accomplish in our Jinja templates, it's important to call out that there are a few common expressions that are used throughout all Jinja templates, including those related to network devices. They are outlined below.

`Comments:`

Comments are represented as such, with our friend the pound symbol:

```jinja
{# Automation Is Fun #}
```

`Expressions/Variables`

Expressions or variables are represented with a pair of curly brackets:

```jinja
 {{ inventory_hostname }}
```

`Statements`

Statements are represented with a percent symbol:

```jinja
{% for items in vars_file['interfaces'] %}
{% endfor %}
```

???+ tip "Live Jinja Parser"
    Quick and easy method for testing data models and Jinja syntax: [https://j2live.ttl255.com/](https://j2live.ttl255.com/ "Live Jinja Parser!")

### Inventory File and Ansible Playbook

While we won't cover the inventory file or Ansible playbooks in depth in this section as it will be covered in Ansible, it is important to call out it's importance in relation to Jinja. You may wonder, when using Ansible to automate and render any configurations, how does it know what devices I want to create configurations for? This is accomplished with the Ansible inventory file. This will be covered more thoroughly, however, for the purpose of the following examples, we will assume we have an inventory file with four hosts, `spine1`, `spine2`, `leaf1`, and `leaf2`. Some examples may show output for all four devices, and some may show output for just one, depending on the complexity of the example. Also, every example we show will use the following Ansible Playbook so you understand the destination filename syntax. There are more parts to this playbook but they will be shown at the end and covered in the next presentation:

```yaml
---
# Inventory File
fabric:
  spines:
    spine1:
    spine2:
  leafs:
    leaf1:
    leaf2:
```

```yaml
---
#Config Playbook
- hosts: spine1,spine2,leaf1,leaf2
  gather_facts: false
  - name: Register variables
    include_vars:
      file: "{{lookup('env','PWD')}}/vars/global.yml"
      name: global
  - name: Register variables
    include_vars:
      file: "{{lookup('env','PWD')}}/vars/interface.yml"
      name: interface
  - name: Create configs
    template:
      src: "{{lookup('env','PWD')}}/templates/full_config.j2"
      dest: "{{lookup('env','PWD')}}/configs/full_config/{{inventory_hostname}}_config.cfg"
```

### Variable Substitution

As shown previously, we know expressions or variable substitution is performed with the double curly brackets, `{{ my_var }}`, but what does this look like in a Jinja template?

For example, we may want to generate the hostname in our template for all the devices in our inventory file. In order to do this we can use a standard Ansible variable called `inventory_hostname`, which substitutes the name of the current inventory host the Ansible play is running against.

```jinja
{# Create a file assigning the device hostname #}

hostname {{ inventory_hostname }}
```

Assuming our pre-defined inventory file, running Ansible against this template with the relevant YAML file called would yield four different output files:

```jinja
spine1_config.cfg
spine2_config.cfg
leaf1_config.cfg
leaf2_config.cfg
```

The output of one of these files all listed below would be as follows:

```jinja
#spine1_config.cfg
hostname spine1

#spine2_config.cfg
hostname spine2

#leaf1_config.cfg
hostname leaf1

#leaf2_config.cfg
hostname leaf2
```

How about a more complex variable substitution using something from one of our data models above. This shows how to substitute for a single dictionary item:

```yaml
# Global.yml
aaa_authentication:
  login:
    default: group radius local

aaa_authorization:
  exec:
    default: group radius local

clock:
  timezone: "America/Detroit"
```

The Jinja template to call these variables is shown here:

```jinja
# Render aaa authC config line
aaa authentication login default {{ global['aaa_authentication']['login']['default'] }}

# Render aaa authZ config line
aaa authorization exec default {{ global['aaa_authorization']['exec']['default'] }}

# Render clock timezone config line
clock timezone {{ global['clock']['timezone']}}
```

As you can see, the variable has many parameters in them. Let's walk through these parameters using the `aaa authentication` config line.

`global`:  Global is the name of the vars file when we registered it in our Ansible playbook. You can register this as anything you want.

`aaa_authentication`:  This is the name of the parent or top level label or key in our dictionary. This tells the Jinja template the next level to look for the variable we substitute.

`login`: This is another label or key down the dictionary, and again tell us where to keep looking for the final variable.

`default`:  This is the key-value pair mapping in the dictionary that we want assigned as the variable. We see this is the final parameter in our variable substitution because we want the value of that key to be the result.

Running the playbook generates the following configuration against all devices called in the inventory file:

??? eos-config annotate "Configuration Output"

    ```yaml
    # AAA authC Login
    aaa authentication login default group radius local

    # AAA authZ Login
    aaa authorization exec default group radius local

    # Device Timezone
    clock timezone America/Detroit
    ```

### Conditionals and Loops

#### Conditionals

Conditionals, such as `{% if %}`, `{% elif %}`, and `{% else %}`, as well as `{% for %}` loops are extremely helpful for either configurations that may apply to only a subset of devices you are generating configurations for. Additionally, for loops are a must to efficiently work through nested data structures like lists of lists or lists of dictionaries.

Let's start our statements journey with conditionals, and where they can be helpful.

In this first example we will look at the following portion of our `interfaces.yml` YAML vars file:

```yaml
---
spine1:
  mlag_side: A
  interfaces:
    Ethernet47:
      desc: To_SPINE2_MLAG_PEERLINK
      mlag_peerlink: true
    Ethernet48:
      desc: To_SPINE2_MLAG_PEERLINK
      mlag_peerlink: true
    Ethernet1:
      desc: TO_LEAF1
      mlag_peerlink: false
    Ethernet2:
      desc: TO_LEAF2
      mlag_peerlink: false
spine2:
  mlag_side: B
  interfaces:
    Ethernet47:
      desc: 'To_SPINE1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet48:
      desc: 'To_SPINE1_MLAG_PEERLINK'
      mlag_peerlink: true
    Ethernet1:
      desc: 'TO_LEAF1'
      mlag_peerlink: false
    Ethernet2:
      desc: 'TO_LEAF2'
      mlag_peerlink: false
```

The following dictionary key is what we are going to pay close attention to for our Jinja template: `mlag_side`

Here we will use a Jinja template to create a partial, correct MLAG configuration per spine device, based on which side it is.

```jinja
{% if interface[inventory_hostname]['mlag_side'] == 'A' %}
int vlan 4094
ip address 192.0.0.0/31

mlag configuration
domain-id workshop
local-interface vlan4094
peer-link po2000
peer-address 192.0.0.1

{% else %}
int vlan 4094
ip address 192.0.0.1/31

mlag configuration
domain-id workshop
local-interface vlan4094
peer-link po2000
peer-address 192.0.0.0

{% endif %}
```

Reviewing this template, we can see we are using a conditional based on which MLAG side the device is. If the device belongs to side **A**, it will apply the matching configuration. If the device belongs to side **B** it will apply the other configuration.

Here is the output of running the Ansible playbook and the configs that are generated:

??? eos-config annotate "spine1 Output"

    ```yaml
    int vlan 4094
    ip address 192.0.0.0/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.1
    ```

??? eos-config annotate "spine2 Output"

    ```yaml
    int vlan 4094
    ip address 192.0.0.1/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.0
    ```

Another aspect of conditionals to match on is a boolean, whether something is true or false, using the following example, also referencing our `interfaces.yml`:

```jinja
{% for item in interface[inventory_hostname]['interfaces'] %}
{% if interface[inventory_hostname]['interfaces'][item]['mlag_peerlink'] == true %}
interface {{ item }}
description {{ interface[inventory_hostname]['interfaces'][item]['desc'] }}
channel-group 2000 mode active
{% else %}

interface {{ item }}
description {{ interface[inventory_hostname]['interfaces'][item]['desc'] }}

{% endif %}
{% endfor %}
```

Ignoring for a moment the for loop part we haven't covered yet, we can see we are checking each interface that is defined to see if the interface parameter for `mlag_peerlink` is set to true, if it is, we want to apply an extra line of configuration, `channel-group 2000 mode active`, if it's not, we just configure the specified interface with a description. The output of running this against the spines would be as follows:

??? eos-config annotate "spine1 Output"

    ```yaml
    interface Ethernet47
    description To_SPINE2_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_SPINE2_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet1
    description TO_LEAF1

    interface Ethernet2
    description TO_LEAF2
    ```

??? eos-config annotate "spine2 Output"

    ```yaml
    interface Ethernet47
    description To_SPINE1_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_SPINE1_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet1
    description TO_LEAF1

    interface Ethernet2
    description TO_LEAF2
    ```

### Loops

Another great function that Jinja templates support is the use of `for` loops. For loops come in handy when trying to iterate through a list of dictionaries to repeat configuration lines.

Let's start with an example using a `for` loop to iterate through a list. For example, the below list shows a list with a single DNS server:

```yaml
# DNS Servers
name_servers:
  - 10.100.100.20
```

The template with a for loop to iterate through this list would be as follows:

```jinja
{% for dns in global['name_servers'] %}
ip name-server {{ dns }}
{% endfor %}
```

Let's analyze the sections of this template.

`dns`:  DNS is a variable that we are using to represent each item in the list. This can be anything you wish.

`global`:  If you recall, this is the name of the YAML file we are registering in our playbook. This tells the template which YAML file to look at for the variable.

`name_servers`:  This is the parent label or key of the list. This says which items in the list we want to iterate through and assign to the variable we defined.

After assigning the value in the list to our created variable, we issue our configuration line which contains static text and our variable. The output would look as follows:

```yaml
ip name-server 10.100.100.20
```

What if we had multiple items in the list like this:

```yaml
# DNS Servers
name_servers:
  - 10.100.100.20
  - 8.8.8.8
  - 4.4.4.4
  - 208.67.222.222
```

The for loop would run as many times as there are items in the list and the configuration file output would look as follows:

```yaml
ip name-server 10.100.100.20
ip name-server 8.8.8.8
ip name-server 4.4.4.4
ip name-server 208.67.222.222
```

Now let's take a look at a slightly more complex, nested data structure, such as a dictionary with a list item. We will use the following portion from our data model:

```yaml
# radius servers
radius_servers:
  - host: 192.168.1.10
    vrf: MGMT
    key: radiusserverkey
```

The template with a for loop to iterate through this list would be as follows:

```jinja
{% for rsrv in global['radius_servers'] %}
radius-server host {{ rsrv['host'] }} vrf {{ rsrv['vrf'] }} key {{ rsrv['key'] }}
{% endfor %}
```

Let's analyze the sections of this template.

`rsrv`:  This is a variable we are setting that represents each item in the list of the dictionary **`radius_servers`**.

`global`: This tells the template which YAML file to look at for the variable.

Looking at the configuration line we created, we see instead of walking through the dictionary via the dictionary key names, we key off our variable which represents the items in our dictionary. This can be seen with the `rsrv['hosts']` line. This means we are looking for the value of the **`host`** key for each server in our list that is currently assigned to the **`rsrv`** variable. The same holds true for the `rsrv['vrf']` and `rsrv['key']` lines.

The for loop would run as many times as there are items in the list, which is just one, and the configuration file output would look as follows:

```yaml
radius-server host 192.168.1.10 vrf MGMT key radiusserverkey
```

While that was simple, what if we have something more complex, like a dictionary with a list of dictionaries? Let's take a look at how this would be represented in Jinja using the following YAML data model from our `global.yml` file.

```yaml
ntp:
  servers:
    - name: 0.north-america.pool.ntp.org
      vrf: MGMT
    - name: 1.north-america.pool.ntp.org
      vrf: MGMT
    - name: 2.north-america.pool.ntp.org
      vrf: MGMT
    - name: time.google.com
      vrf: MGMT
```

The template with a for loop to iterate through this list would be as follows:

```jinja
{% for ntps in global['ntp']['servers'] %}
ntp server vrf {{ ntps['vrf'] }} {{ ntps['name'] }}
{% endfor %}
```

Let's analyze the sections of this template.

`ntps`:  This is another variable we are setting that represents each item in the list of the dictionary **servers**.

`global`: This tells the template which YAML file to look at for the variable.

Looking at the configuration line we created, we see instead of walking through the dictionary via the dictionary key names, we key off our variable which represents the items in our dictionary. This can be seen with the `ntps['vrf']` line. This means we are looking for the value of the **`vrf`** key for each server in our list that is currently assigned to the **`ntps`** variable. The same holds true for the `ntps['name']` line.

The for loop would run as many times as there are items in the list and the configuration file output would look as follows:

```yaml
ntp server vrf MGMT 0.north-america.pool.ntp.org
ntp server vrf MGMT 1.north-america.pool.ntp.org
ntp server vrf MGMT 2.north-america.pool.ntp.org
ntp server vrf MGMT time.google.com
```

In the previous examples we only covered single for loops iterating a single layer deep, however, what if we have a dictionary within a dictionary, which contains a list? In that instance we would need to have nested for loops in our templates. For this example we will introduce a new YAML data model to better illustrate nested for loops. This section shows some configuration you may need to apply related to EVPN VXLAN:

```yaml
---
# EVPN VXLAN vrfs
Red:
  l3vni: 10000
  vlans:
    101:
      l2vni: 10001
      anycast_gw: 10.10.10.1/24
    102:
      l2vni: 10002
      anycast_gw: 10.10.20.1/24
Blue:
  l3vni: 20000
  vlans:
    201:
      l2vni: 20001
      anycast_gw: 10.20.10.1/24
    202:
      l2vni: 20002
      anycast_gw: 10.20.20.1/24
```

Reviewing this data model we can see we have two dictionaries that represent different VRFs, **`Red`** and **`Blue`**. Within each of those dictionaries we have another dictionary called **`vlans`**, which defines the interfaces in each of those VRFs. When performing the configuration of a VXLAN interface we need to define both the l3vni, as well as each VLAN to VXLAN mapping. Then we need to loop through each of the parent keys followed by each of the items in **`vlans`** dictionary.

The template to accomplish this would look like this:

```jinja
{# Leaf evpn config #}
interface vxlan1
vxlan source-interface loopback1
vxlan udp-port 4789

{% for vrf,vrf_values in vrf.items() %}

vxlan vrf {{vrf}} vni {{ vrf_values['l3vni']}}

{% for vlan,vlan_values in vrf_values['vlans'].items() %}

vxlan vlan {{ vlan }} vni {{ vlan_values['l2vni'] }}

{% endfor %}
{% endfor %}
```

Looking at this template, let's walk through what's happening. First, we have a for loop that is iterating through the top level dictionary keys in the YAML file itself. Those keys would be `Red` and `Blue`. Looking at the actual for loop syntax itself, we have some new parameters to look at:

`vrf`:  This is a variable the keys in the dictionary list are assigned to.

`vrf_values`:  This is another variable the corresponding values of those keys are assigned to.

`vrf.items()`: This uses the built in .items function to determine which items in the dictionary we are looping through. It includes the top level keys.

Using those parameters, we construct our configuration line, which includes the variable `vrf`, the key, and the value for the pair `l3vni`.

After this we repeat the same loop logic, but for the nested dictionary. In this instance we create a new set of variables for the keys and values, only this time, those are assigned the values within the `vlans` dictionary, as called by the `vrf_values['vlans'].items()` parameter.

Before we view the final output, let's look at what the variable output actually is. This will both make it clearer what they represent based on the data model, as well as show you a nice way to troubleshoot your template while you are working on it.

First we will look at the `vrf` variables, and we will do this by changing the template to this:

```jinja
{# Leaf evpn config #}
{# interface vxlan1
vxlan source-interface loopback1
vxlan udp-port 4789 #}
{% for vrf,vrf_values in vrf.items() %}
{# vxlan vrf {{vrf}} vni {{ vrf_values['l3vni']}}  #}
{{ vrf }}

{% for vlan,vlan_values in vrf_values['vlans'].items() %}
{# vxlan vlan {{ vlan }} vni {{ vlan_params['l2vni'] }} #}
{{ vlan }}

{% endfor %}
{% endfor %}
```

Reviewing the above template, we will comment out everything except our for loop and our initial key variables. The result of this template will run through the nested loops, and output the value of the variables called, `{{ vrf }}`, and `{{ vlan }}`.

Viewing the output, we can see that these equal the top level keys of our data model:

```yaml
Red

101

102


Blue

201

202
```

The first for loop runs and you can see `vrf=Red`. Then the nested loop runs twice due to there being two VLANs, and we can see both VLAN values. This then repeats for the second top level key, `Blue`.

Now let's take a look at the `_params` variable. Changing the template to what follows and commenting out our initial variable, and adding a substitution line for the `_params` variable:

```jinja
{# Leaf evpn config #}
{# interface vxlan1
vxlan source-interface loopback1
vxlan udp-port 4789 #}
{% for vrf,vrf_values in vrf.items() %}
{# vxlan vrf {{vrf}} vni {{ vrf_values['l3vni']}}  #}
{# {{ vrf }} #}
{{ vrf_values }}

{% for vlan,vlan_values in vrf_values['vlans'].items() %}
{# vxlan vlan {{ vlan }} vni {{ vlan_params['l2vni'] }} #}
{# {{ vlan }} #}
{{ vlan_values }}

{% endfor %}
{% endfor %}
```

Viewing the output, we can see that these equal the dictionary key-pair values of the nested `vlan` dictionary:

```yaml
{'l3vni': 10000, 'vlans': {101: {'l2vni': 10001, 'anycast_gw': '10.10.10.1/24'}, 102: {'l2vni': 10002, 'anycast_gw': '10.10.20.1/24'}}}

{'l2vni': 10001, 'anycast_gw': '10.10.10.1/24'}

{'l2vni': 10002, 'anycast_gw': '10.10.20.1/24'}


{'l3vni': 20000, 'vlans': {201: {'l2vni': 20001, 'anycast_gw': '10.20.10.1/24'}, 202: {'l2vni': 20002, 'anycast_gw': '10.20.20.1/24'}}}

{'l2vni': 20001, 'anycast_gw': '10.20.10.1/24'}

{'l2vni': 20002, 'anycast_gw': '10.20.20.1/24'}
```

Reverting back to our desired template, we can see what the template and actual output would be:

```jinja
{# Leaf evpn config #}
interface vxlan1
vxlan source-interface loopback1
vxlan udp-port 4789

{% for vrf,vrf_values in vrf.items() %}

vxlan vrf {{vrf}} vni {{ vrf_values['l3vni']}}

{% for vlan,vlan_values in vrf_values['vlans'].items() %}

vxlan vlan {{ vlan }} vni {{ vlan_values['l2vni'] }}

{% endfor %}
{% endfor %}
```

```yaml
interface vxlan1
vxlan source-interface loopback1
vxlan udp-port 4789
vxlan vrf Red vni 10000
vxlan vlan 101 vni 10001
vxlan vlan 102 vni 10002
vxlan vrf Blue vni 20000
vxlan vlan 201 vni 20001
vxlan vlan 202 vni 20002
```

### Filters

The final section we will cover will be filters. While there are an enormous amount of filters available, we will just cover a very common one, the `ipaddr` filter. In a simple explanation, the `ipaddr` filter takes a full IP address and subnet mask, and strips off just the mask, with the end result being only the address. This can be helpful in an instance where you are using a full prefix and mask in your data model and don't want to create a new, duplicate, key-value pair mapping to be called.

#### IPADDR()

In a simple explanation, the `ipaddr` filter has various operations allowing you to manipulate a prefix or network and obtain certain information about it. This can be helpful in an instance where you are using a full prefix and mask in your data model and don't want to create a new, duplicate, key-value pair mapping to be called.

The following are some of the functions available within the **ipaddr()** filter:

`address:` When using a prefix in x.x.x.x/yy notation, this filter will pull only the address portion.

`network:` This filter will calculate the network ID of a given prefix.

`netmask:` This filter will generate the subnet mask from /yy prefix/CIDR notation.

`broadcast:` This filter will calculate the broadcast address of a given prefix.

`size:` This filter will calculate the size of a subnet based on the subnet mask.

`range_usable:` This filter finds the range of usable addresses within a subnet.

To illustrate this, we will use a simple example for showing the IP info of these two interfaces run against a single device, **spine1**.

The data model is as follows:

```yaml
---
spine1:
  interfaces:
    Ethernet1:
      ipv4: 1.1.1.1/26
    Ethernet2:
      ipv4: 2.2.2.2/8
```

The Jinja template is as follows, which will use the above described filters to give us various information about the IP addresses assigned to Ethernet1 and Ethernet2:

```jinja
Here is some information about some of Spine1's interfaces:

Ethernet1
    IP: {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('address')  }}
    Subnet Mask:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('netmask')  }}
    Network ID:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('network')  }}
    Network Size:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('size')  }}
    Usable Range:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('range_usable')  }}
    Broadcast Address:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet1']['ipv4'] | ipaddr('broadcast')  }}

Ethernet2
    IP: {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('address')  }}
    Subnet Mask:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('netmask')  }}
    Network ID:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('network')  }}
    Network Size:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('size')  }}
    Usable Range:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('range_usable')  }}
    Broadcast Address:  {{ ip_filter[inventory_hostname]['interfaces']['Ethernet2']['ipv4'] | ipaddr('broadcast')  }}
```

Reviewing our template, we run the playbook against the current inventory device keying in on the ipv4 address mapping. We then use the filter command, `|`, and specify the `address` keyword, meaning we only want the address part of the whole prefix.

Running the playbook results in the following output:

```yaml
Here is some information about some of Spine1's interfaces:

Ethernet1
    IP: 1.1.1.1
    Subnet Mask:  255.255.255.192
    Network ID:  1.1.1.0
    Network Size:  64
    Usable Range:  1.1.1.1-1.1.1.62
    Broadcast Address:  1.1.1.63

Ethernet2
    IP: 2.2.2.2
    Subnet Mask:  255.0.0.0
    Network ID:  2.0.0.0
    Network Size:  16777216
    Usable Range:  2.0.0.1-2.255.255.254
    Broadcast Address:  2.255.255.255
```

#### JOIN()

When working with lists in YAML, it may be needed to have all items in a list concatenated into a single line of a configuration command, instead of looping through the list and creating a configuration command for each item. Some examples of this could be configuring a list of NTP or DNS servers in a single line, versus individual entries. This can be accomplished with the following filter:

`join(" "):` This filter joins all the items in the list. Pay close attention to the space between the double quotes. This provides spacing between the items in a list. Without this space, all items in the list would be joined into one continuous string.

To illustrate this, we will use simple DNS server data model and Jinja template.

The data model is as follows:

```yaml
---
# DNS Servers
name_servers:
  - 10.100.100.20
  - 8.8.8.8
  - 4.4.4.4
  - 208.67.222.222
```

The following Jinja template will allow us to configure all four DNS servers on a single line for a single configuration command:

```jinja
ip name-server {{ global['name_servers'] | join(" ") }}
```

Running the playbook results in the following output:

```yaml
ip name-server 10.100.100.20 8.8.8.8 4.4.4.4 208.67.222.222
```

## The Jinja YAML Relationship

As has hopefully been explained well, the interdependent relationship between Jinja and YAML is that Jinja templates utilize the YAML vars files and their data model to generate new configurations, whether full or partial. YAML files serve no purpose without being called and their variables used, and configurations rendered from Jinja templates would be no different than static configs without the variable substitution of YAML files.

## Jinja Templates

### Network Config Template Example

Using all the above sections we have reviewed, let's put it all together into a template called `full_config.j2`, which renders an entire device config based on the `global.yml` and `interface.yml` YAML files.

The Jinja template would look as follows:

??? eos-config annotate "full_config.j2"

    ```jinja
    {# Full Config #}

    hostname {{ inventory_hostname }}

    username admin priv {{ global['local_users']['admin']['privilege'] }} role {{ global['local_users']['admin']['role'] }} secret {{ global['local_users']['admin']['secret'] }}

    username noc priv {{ global['local_users']['noc']['privilege'] }} role {{ global['local_users']['noc']['role'] }} secret {{ global['local_users']['noc']['secret'] }}

    aaa authentication login default {{ global['aaa_authentication']['login']['default'] }}

    aaa authorization exec default {{ global['aaa_authorization']['exec']['default'] }}

    {% for rsrv in global['radius_servers'] %}
    radius-server host {{ rsrv['host'] }} vrf {{ rsrv['vrf'] }} key {{ rsrv['key'] }}
    {% endfor %}

    {% for httpsrc in global['ip_http_client_source_interfaces'] %}
    ip http client local-interface {{ httpsrc['name'] }} vrf {{ httpsrc['vrf'] }}
    {% endfor %}

    {% for radsrc in global['ip_radius_source_interfaces'] %}
    ip radius vrf {{ radsrc['vrf'] }} source-interface {{ radsrc['name'] }}
    {% endfor %}

    mac address-table aging-time {{ global['mac_address_table']['aging_time']}}

    arp aging timeout {{ global['arp']['aging']['timeout_default']}}

    {% for dns in global['name_servers'] %}
    ip name-server {{ dns }}
    {% endfor %}

    ip domain lookup vrf {{ global['ip_domain_lookup']['source_interfaces']['Management1']['vrf'] }} source-interface Management1


    {% for ntps in global['ntp']['servers'] %}
    ntp server vrf {{ ntps['vrf'] }} {{ ntps['name'] }}
    {% endfor %}

    clock timezone {{ global['clock']['timezone']}}

    vlan 4094
    name MLAG
    trunk group MLAGPeer

    no spanning-tree vlan 4094

    {% for item in interface[inventory_hostname]['interfaces'] %}
    {% if interface[inventory_hostname]['interfaces'][item]['mlag_peerlink'] == true %}
    interface {{ item }}
    description {{ interface[inventory_hostname]['interfaces'][item]['desc'] }}
    channel-group 2000 mode active
    {% else %}

    interface {{ item }}
    description {{ interface[inventory_hostname]['interfaces'][item]['desc'] }}

    {% endif %}

    {% endfor %}

    interface port-channel 2000
    switchport
    switchport mode trunk
    switchport trunk group MLAGPeer

    {% if interface[inventory_hostname]['mlag_side'] == 'A' %}
    int vlan 4094
    ip address 192.0.0.0/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.1

    {% else %}

    int vlan 4094
    ip address 192.0.0.1/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.0
    {% endif %}
    ```

# Final Output - Tying It All Together

Let's put our full YAML data models below, as well as the output that is generated for `spine1-2` and `leaf1-2`:

??? eos-config annotate "global.yml"

    ```yaml
    # local users
    local_users:
      admin:
        privilege: 15
        role: network-admin
        secret: aristaadmin
      noc:
        privilege: 1
        role: network-operator
        secret: aristaops

    # aaa authentication and authorization
    aaa_authentication:
      login:
        default: group radius local

    aaa_authorization:
      exec:
        default: group radius local

    # radius servers
    radius_servers:
      - host: 192.168.1.10
        vrf: MGMT
        key: radiusserverkey

    # HTTP Client source interface and VRF
    ip_http_client_source_interfaces:
        - name: Management1
          vrf: MGMT

    # RADIUS source interface and VRF
    ip_radius_source_interfaces:
      - name: Management1
        vrf: MGMT

    #MAC and ARP aging timers
    mac_address_table:
      aging_time: 1800

    arp:
      aging:
        timeout_default: 1500

    # DNS Servers
    name_servers:
      - 10.100.100.20
      - 8.8.8.8
      - 4.4.4.4
      - 208.67.222.222

    # DNS lookup source interface (Servers defined in 1L2P.yml)
    ip_domain_lookup:
      source_interfaces:
        Management1:
          vrf: MGMT

    # NTP Servers (source interface defined in group specific YML files (CORE, ACCESS, MGMT, INET)
    ntp:
      local_interface:
        name: Management1
        vrf: MGMT
      servers:
        - name: 0.north-america.pool.ntp.org
          vrf: MGMT
        - name: 1.north-america.pool.ntp.org
          vrf: MGMT
        - name: 2.north-america.pool.ntp.org
          vrf: MGMT
        - name: time.google.com
          vrf: MGMT

    clock:
      timezone: "America/Detroit"
    ```

??? eos-config annotate "interface.yml"

    ```yaml
    ---
    spine1:
      mlag_side: A
      interfaces:
        Ethernet47:
          desc: To_SPINE2_MLAG_PEERLINK
          mlag_peerlink: true
        Ethernet48:
          desc: To_SPINE2_MLAG_PEERLINK
          mlag_peerlink: true
        Ethernet1:
          desc: TO_LEAF1
          mlag_peerlink: false
        Ethernet2:
          desc: TO_LEAF2
          mlag_peerlink: false
    spine2:
      mlag_side: B
      interfaces:
        Ethernet47:
          desc: 'To_SPINE1_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet48:
          desc: 'To_SPINE1_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet1:
          desc: 'TO_LEAF1'
          mlag_peerlink: false
        Ethernet2:
          desc: 'TO_LEAF2'
          mlag_peerlink: false
    leaf1:
      mlag_side: A
      interfaces:
        Ethernet47:
          desc: 'To_LEAF2_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet48:
          desc: 'To_LEAF2_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet1:
          desc: 'TO_SPINE1'
          mlag_peerlink: false
        Ethernet2:
          desc: 'TO_SPINE2'
          mlag_peerlink: false
    leaf2:
      mlag_side: B
      interfaces:
        Ethernet47:
          desc: 'To_LEAF1_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet48:
          desc: 'To_LEAF1_MLAG_PEERLINK'
          mlag_peerlink: true
        Ethernet1:
          desc: 'TO_SPINE1'
          mlag_peerlink: false
        Ethernet2:
          desc: 'TO_SPINE2'
          mlag_peerlink: false
    ```

And the configuration outputs:

??? eos-config annotate "spine1_config.cfg"

    ```yaml
    hostname spine1

    username admin priv 15 role network-admin secret aristaadmin

    username noc priv 1 role network-operator secret aristaops

    aaa authentication login default group radius local

    aaa authorization exec default group radius local

    radius-server host 192.168.1.10 vrf MGMT key radiusserverkey

    ip http client local-interface Management1 vrf MGMT

    ip radius vrf MGMT source-interface Management1

    mac address-table aging-time 1800

    arp aging timeout 1500

    ip name-server 10.100.100.20
    ip name-server 8.8.8.8
    ip name-server 4.4.4.4
    ip name-server 208.67.222.222

    ip domain lookup vrf MGMT source-interface Management1


    ntp server vrf MGMT 0.north-america.pool.ntp.org
    ntp server vrf MGMT 1.north-america.pool.ntp.org
    ntp server vrf MGMT 2.north-america.pool.ntp.org
    ntp server vrf MGMT time.google.com

    clock timezone America/Detroit

    vlan 4094
    name MLAG
    trunk group MLAGPeer

    no spanning-tree vlan 4094

    interface Ethernet47
    description To_SPINE2_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_SPINE2_MLAG_PEERLINK
    channel-group 2000 mode active


    interface Ethernet1
    description TO_LEAF1



    interface Ethernet2
    description TO_LEAF2



    interface port-channel 2000
    switchport
    switchport mode trunk
    switchport trunk group MLAGPeer

    int vlan 4094
    ip address 192.0.0.0/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.1
    ```

??? eos-config annotate "spine2_config.cfg"

    ```yaml
    hostname spine2

    username admin priv 15 role network-admin secret aristaadmin

    username noc priv 1 role network-operator secret aristaops

    aaa authentication login default group radius local

    aaa authorization exec default group radius local

    radius-server host 192.168.1.10 vrf MGMT key radiusserverkey

    ip http client local-interface Management1 vrf MGMT

    ip radius vrf MGMT source-interface Management1

    mac address-table aging-time 1800

    arp aging timeout 1500

    ip name-server 10.100.100.20
    ip name-server 8.8.8.8
    ip name-server 4.4.4.4
    ip name-server 208.67.222.222

    ip domain lookup vrf MGMT source-interface Management1


    ntp server vrf MGMT 0.north-america.pool.ntp.org
    ntp server vrf MGMT 1.north-america.pool.ntp.org
    ntp server vrf MGMT 2.north-america.pool.ntp.org
    ntp server vrf MGMT time.google.com

    clock timezone America/Detroit

    vlan 4094
    name MLAG
    trunk group MLAGPeer

    no spanning-tree vlan 4094

    interface Ethernet47
    description To_SPINE1_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_SPINE1_MLAG_PEERLINK
    channel-group 2000 mode active


    interface Ethernet1
    description TO_LEAF1



    interface Ethernet2
    description TO_LEAF2



    interface port-channel 2000
    switchport
    switchport mode trunk
    switchport trunk group MLAGPeer

    int vlan 4094
    ip address 192.0.0.1/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.0
    ```

??? eos-config annotate "leaf1_config.cfg"

    ```yaml
    hostname leaf1

    username admin priv 15 role network-admin secret aristaadmin

    username noc priv 1 role network-operator secret aristaops

    aaa authentication login default group radius local

    aaa authorization exec default group radius local

    radius-server host 192.168.1.10 vrf MGMT key radiusserverkey

    ip http client local-interface Management1 vrf MGMT

    ip radius vrf MGMT source-interface Management1

    mac address-table aging-time 1800

    arp aging timeout 1500

    ip name-server 10.100.100.20
    ip name-server 8.8.8.8
    ip name-server 4.4.4.4
    ip name-server 208.67.222.222

    ip domain lookup vrf MGMT source-interface Management1


    ntp server vrf MGMT 0.north-america.pool.ntp.org
    ntp server vrf MGMT 1.north-america.pool.ntp.org
    ntp server vrf MGMT 2.north-america.pool.ntp.org
    ntp server vrf MGMT time.google.com

    clock timezone America/Detroit

    vlan 4094
    name MLAG
    trunk group MLAGPeer

    no spanning-tree vlan 4094

    interface Ethernet47
    description To_LEAF2_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_LEAF2_MLAG_PEERLINK
    channel-group 2000 mode active


    interface Ethernet1
    description TO_SPINE1



    interface Ethernet2
    description TO_SPINE2



    interface port-channel 2000
    switchport
    switchport mode trunk
    switchport trunk group MLAGPeer

    int vlan 4094
    ip address 192.0.0.0/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.1

    ```

??? eos-config annotate "leaf2_config.cfg"

    ```yaml
    hostname leaf2

    username admin priv 15 role network-admin secret aristaadmin

    username noc priv 1 role network-operator secret aristaops

    aaa authentication login default group radius local

    aaa authorization exec default group radius local

    radius-server host 192.168.1.10 vrf MGMT key radiusserverkey

    ip http client local-interface Management1 vrf MGMT

    ip radius vrf MGMT source-interface Management1

    mac address-table aging-time 1800

    arp aging timeout 1500

    ip name-server 10.100.100.20
    ip name-server 8.8.8.8
    ip name-server 4.4.4.4
    ip name-server 208.67.222.222

    ip domain lookup vrf MGMT source-interface Management1


    ntp server vrf MGMT 0.north-america.pool.ntp.org
    ntp server vrf MGMT 1.north-america.pool.ntp.org
    ntp server vrf MGMT 2.north-america.pool.ntp.org
    ntp server vrf MGMT time.google.com

    clock timezone America/Detroit

    vlan 4094
    name MLAG
    trunk group MLAGPeer

    no spanning-tree vlan 4094

    interface Ethernet47
    description To_LEAF1_MLAG_PEERLINK
    channel-group 2000 mode active

    interface Ethernet48
    description To_LEAF1_MLAG_PEERLINK
    channel-group 2000 mode active


    interface Ethernet1
    description TO_SPINE1



    interface Ethernet2
    description TO_SPINE2



    interface port-channel 2000
    switchport
    switchport mode trunk
    switchport trunk group MLAGPeer

    int vlan 4094
    ip address 192.0.0.1/31

    mlag configuration
    domain-id workshop
    local-interface vlan4094
    peer-link po2000
    peer-address 192.0.0.0
    ```
