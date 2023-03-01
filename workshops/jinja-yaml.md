![Jinja YAML Banner](assets/images/jinja-yaml-banner.png)

# **Welcome to YAML & Jinja**

This section will cover both Jinja and YAML, which are two interdependent pieces of basic configuration automation framework.  While both YAML and Jinja can get relatively complex with what they can accomplish and what can be done with them, we will only cover what is necessary to utilize these tools for network automation and DevOps.  At the end of the YAML and JINJA section, we will tie everything together with a final output.

# **YAML**

## **What is YAML?**

YAML is what's officially referred to as a data serialization language.  While that sounds complex, data serialization is simply the process of converting objects within a data model into a byte stream for the purpose of storing or transferring it.  Breaking this down, we know when trying to automate configuration management or deployment, we need to specify ***what*** it is we want to configure, that is, what should our desired end state configuration look like.  ( **intent based networking** )

<br>

While this workshop is geared towards network automation, YAML files are not restricted to only representing a data model of a network configuration.  YAML files can be created to represent any parameters or information about anything you wish.  In fact, to help ease the learning of YAML files and data models, while reviewing all the different pieces that can be used in a YAML file, there will be two examples for each section.  One example will reference applicable sections of network device configurations, something you would encounter and see during your job, while the other example will use something completely unrelated, for the purpose of abstracting the point of YAML.

<br>

A final, important note is about why we are talking about and using YAML in the first place.  As mentioned in the beginning, YAML is a data serialization langauge, however, it is not the only one.  Some other common, data serialization languages are XML, JSON, and CSV.  The reason we are particular towards YAML is that not only are there libraries available in most programming languages, but also because as the following table shows, it is very human readable.

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

<br>

## **Common YAML Syntax**

Now that we know the idea of what YAML is and what it's used for, we can cover the various constructs within a YAML file, and how to use those expressions to create files that are useful to us as engineers trying to manage configurations.

### **Comments**

Comments are an important part of any documentation or configuration you are doing.  When creating or modify any network configurations, while you don't realize it, you are commenting something that is important.  When configuring an interface, you may add a description, such as the name of a downstream device or server it's connected to.  Or if its a provider handoff you may add the NOC telephone number and the circuitID.  If you are configuring BGP peers, you may add a description to each peer so it's clear what the neighbor is.

<br>

In YAML, you can enter comments anywhere you'd like, and ideally, where it makes sense to help explain what your data model is representing.  Comments in YAML are represented with the pound symbol, and shown below.

<br>

```yaml
---
# This is my YAML vars file for global configuration settings.
some config:  here

# Management Interface Configuration
mgmt_gateway: 192.168.0.1
mgmt_interface: Management0
mgmt_interface_vrf: default
```

:writing_hand: **Tip:**  In VSCode, you can auto-comment any text you want by selecting the text and pressing ***windows*** `winkey + /` or ***mac*** `cmd + /` .

<br>

### **Mappings**

The first, and basic YAML construct is a mapping, which comprises of what is technically called a `scalar`.  This will probably be the only time you hear or reference the word **scalar**, ever.

<br>

```yaml
---
# These are some examples of scalars

integer: 10
boolean: true
string: "Welcome to the Automation Workshop"
```

<br>

Sometimes, a scalar is referred to as only the initial part of the line of text such as a list of names or locations like such.

```yaml
---
# Another scalar example

- White Rim Trail
- Hells Revenge
- Long Canyon Road
- Black Bears Pass
- Ophir Pass
```

<br>

In normal day to day use, you will see scalars referred to as **key value pair mappings**, or just **mappings** for short.  A mapping is the data label, followed by its value, separated by a **`:`** colon.

Here are some network config specific mappings:

```yaml
---
# Network Config Mappings

mgmt_interface: Management1
mgmt_interface_vrf: MGMT

spanning_tree_mode: rapid-pvst
spanning_tree_priority: 4096
```

<br>

### **Boolean**

Another type of mapping in YAML are booleans.  Booleans area simply a `true` or `false` value assigned to its data label.  While the actual true or false does not have to be in a certain case, if you want the value to be compatible with lint options, you should use all lowercase.

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

<br>

### **Lists**

Lists, which are sometimes also called sequences, are similar to arrays in development.  A list has a top level label, or what I like to refer to as a key, which represent the elements in the list.  ***( The key nomenclature will come in handy when thinking through loops in Jinja templates )***  In a standard list, you will have singular entries below the top level label.

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

<br>

### **Dictionary**

While we just covered lists, which allow you to specify an item that falls inline below a parent label or key, what happens when you want to include more specific attributes to that parent label, or if you want to introduce more details such as key-value pair mappings to your list?  In that occasion, what is used, and what you will see regularly in just about every network vars file, is a dictionary.  As previously described, a dictionary is a list of key-value mappings.

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

<br>

### **Nested Data Structures**

One of the double edged parts of YAML is that it can be quite complex.  This is good because it is very flexible for how we can build our data model, but it also means that model can get out of control.  YAML syntax is hierarchical, and at the same time, just about every construct we covered previous to this can be nested within each other.  Lets take a look at some nested data structures that illustrate this.

In this first example we will have a list of dictionaries:

:writing_hand: **Reminder:**  Indentation is key.

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

<br>

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
    tags: ['10']
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

<br>

## **YAML File Examples**

Now that we have gone through all the most common constructs within a YAML file used for variables for network device configs, lets see what some complete files would look like for a real world environment.

### **Network Config Example**

These example YAML files could be used to build a base config for a series of devices.  The devices would be based on your inventory file, however, for this example, we will assume there is a layer2 leaf/spine topology, with two spines and two leafs.  For this example we will use two files, a `global.yml` and a `interface.yml` file.

The global.yml file that follows includes the data model used for the base configuration.  These items apply to all four devices in the fabric.  ***( This could be imported in the playbook, or put in the group_vars directory and named after the level of your heirarchy that contains the devices you want this to apply to. )***

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

<br>

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

<br>
<br>
<br>

# **Jinja**

## **What is Jinja?**

At a high level, Jinja is a templating engine used to create markup files such as HTML or XML as well as custom text files, such as in our instance config files.  Under the hood, Jinja is an open source python library that lets you create extensible templates.  One of the major benefits of Jinja is that the template files you create allow you to define both static text, as well as variables.  Some of the Jinja template syntax may look familiar because Jinja is not the first templating engine, and is actually inspired by Django.

<br>

## **What is Jinja Used For?**

As it may have become apparent from the YAML section, after creating our data model of the various configuration parameters we want to automate, we need to get that data model and all its variables into a format that can read and understood by our network devices.  This is where Jinja comes in.  The use of Jinja templates, along with some yet to be shown Ansible magic, allows us to render full or partial configuration files that can loaded onto network devices.  The underlying purpose of this, as it relates to automation, is that with the use of various expressions and variables in the Jinja templates, we can use a single template, with single or multiple YAML variable files, and create configurations against a multitude of network devices.  As far as the actual template file and its file extension, technically any file can be called as a template regardless of its extension as long as its formatted correctly, however, we typically use the `.j2` extension on all Jinja template files.

## **Jinja Syntax**

As a foreward to getting into the different tasks we can accomplish in our Jinja templates, it's important to call out that there are a few common expressions that are used throughout all Jinja templates, including those related to network devices.  They are outlined below.

<br>

`Comments:`

Comments are represented as such, with our friend the pound symbol:

```bash
{# Automation Is Fun #}
```

<br>

`Expressions/Variables`

Expressions or variables are represented with a pair of curly brackets:

```bash
 {{ inventory_hostname }}
 ```

<br>

`Statements`

Statements are represented with a percent symbol:

```bash
{% for items in vars_file['interfaces'] %}
{% endfor %}
```

<br>

### Inventory File and Ansible Playbook

While we won't cover the inventory file or Ansible playbooks in depth in this section as it will be covered in Ansible, it is important to call out it's importance in relation to Jinja.  You may wonder, when using Ansible to automate and render any configurations, how does it know what devices I want to create configurations for?  This is accomplished with the Ansible inventory file.  This will be covered more thoroughly, however, for the purpose of the following examples, we will assume we have an inventory file with four hosts, `spine1`, `spine2`, `leaf1`, and `leaf2`.  Some examples may show output for all four devices, and some may show output for just one, depending on the complexity of the example.  Also, every example we show will use the following Ansible Playbook so you understand the destination filename syntax.  There are more parts to this playbook but they will be shown at the end and covered in the next presentation:

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

<br>

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

### **Variable Substitution**

As shown previously, we know expressions or variable substitution is performed with the double curly brackets, `{{ my_var }}`, but what does this look like in a Jinja template?

For example, we may want to generate the hostname in our template for all the devices in our inventory file.  In order to this we can use a standard Ansible variable called `inventory_hostname`, which substitutes in the current name of the inventory host the Ansible play is running against.

```bash
{# Create a file assigning the device hostname #}

hostname {{ inventory_hostname }}
```

Assuming our pre-defined inventory file, running Ansible against this template with the relevant YAML file called would yield four different output files:

```bash
spine1_config.cfg
spine2_config.cfg
leaf1_config.cfg
leaf2_config.cfg
```

<br>

The output of one of these files all listed below would be as follows:

```bash
#spine1_config.cfg
hostname spine1

#spine2_config.cfg
hostname spine2

#leaf1_config.cfg
hostname leaf1

#leaf2_config.cfg
hostname leaf2
```

<br>

How about a more complex variable substitution using something from one of our data models above.  This shows how to substitute for a single dictionary item:

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

```bash
# Render aaa authC config line
aaa authentication login default {{ global['aaa_authentication']['login']['default'] }}

# Render aaa authZ config line
aaa authorization exec default {{ global['aaa_authorization']['exec']['default'] }}

# Render clock timezone config line
clock timezone {{ global['clock']['timezone']}}
```

<br>

As you can see, the variable has many parameters in them.  Lets walk through these parameters using the `aaa authentication` config line.

`global`:  Global is the name of the vars file when we registered it in our Ansible playbook.  You can register this as anything you want.

`aaa_authentication`:  This is the name of the parent or top level label or key in our dictionary.  This tells the Jinja template the next level to look for the variable we substitute.

`login`: This is another label or key down the dictionary, and again tell us where to keep looking for the final variable.

`default`:  This is the key-value pair mapping in the dictionary that we want assigned as the variable.  We see this is the final parameter in our variable substitution because we want the value of that key to be the result.

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

<br>

### **Conditionals and Loops**

Conditionals, such as `{% if %}`, `{% elif %}`, and `{% else %}`, as well as `{% for %}` loops are extremely helpful for either or configurations that may apply to only a subset of devices you are generating configurations for.  Additionally, for loops are a must for efficiently working through nested data structures like lists of lists or lists of dictionaries.

Lets start our statements journey with conditionals, and where they can be helpful.

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

```bash
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

Reviewing this template, we can see we are using a conditional based on which MLAG side the device is.  If the device belongs to side **A**, it will apply the matching configuration.  If the device belongs to side **B** it will apply the other configuration.

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

### **Filters**

## **Jinja Templates**

### **Network Config Template Example**

<br>
<br>
<br>

# **The Jinja YAML Relationship**

<br>
<br>
<br>

# **Final Output - Tying It All Together**
