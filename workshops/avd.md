# Arista CI Workshop

In this workshop, we will leverage open source tools to build a network CI pipeline for configuration development, deployment, documentation and validation. This will enable us to manage our network environment as code.

This section will cover:

- Arista Validated Designs (AVD) Ansible Collection
- Network Data Models
- Initial Deployment (Day 0 Provisioning)
- Ongoing Operations (Day 2 and Beyond)
- Validation and Troubleshooting
- Enhancing our CI/CD Pipelines with GitHub Actions

Each attendee will receive their own dedicated virtual lab environment with Git, VS Code and Ansible already installed and ready to use.

Attendees will need:

- A laptop
- An account on [GitHub](https://github.com/)
- Familiarity with the concepts and tools covered in the previous [Automation Fundamentals workshop](https://aristanetworks.github.io/avd-workshops/) (Git, VS Code, Jinja/YAML, Ansible)

## Installation Requirements

### ATD Environment

The Lab ATD environment has been pre-installed with Ansible and Git. We still must update AVD and the required modules to the latest version. The following commands will install AVD and needed modules.

``` bash
ansible-galaxy collection install arista.avd arista.cvp --force
export ARISTA_AVD_DIR=$(ansible-galaxy collection list arista.avd --format yaml | head -1 | cut -d: -f1)
pip3 config set global.disable-pip-version-check true
pip3 install -r ${ARISTA_AVD_DIR}/arista/avd/requirements.txt
```

???+ Note

    IMPORTANT: The above steps must be ran each time you start your lab.

### Other Environments

Install AVD and required modules - Installation guide found [here](https://avd.sh/en/stable/docs/installation/collection-installation.html).

## Lab Topology Overview

Throughout this section we will make use of the following dual datacenter topology. Click on the image to zoom in for details.

![Dual DC Topology](assets/images/dual-dc-topo.svg)

## Basic EOS Switch Configuration

Basic connectivity between the Ansible controller host and the switches must be established before Ansible can be used to deploy configurations. The following should be configured on all switches:

- Switch Hostname
- IP enabled interface
- Username and Password defined
- Management eAPI enabled

???+ info

    In the ATD environment, vEOS/cEOS virtual switches use `Management0` in the default VRF. When using actual hardware switches, `Management1` is used. The included basic switch configurations may need to be adjusted for your environment.

Below is an example basic configuration file for s1-spine1:

``` shell
--8<--
workshops/assets/examples/s1-spine1.cfg
--8<--
```

## Ansible Inventory

Our lab L2LS topology contains two sites, `Site 1` and `Site 2`. We need to create the Ansible inventory for each site. We have created two separate directories for each site under the `sites` sub-directory in our repo.

``` text
├── sites/
  ├── site_1/
  ├── site_2/
```

The following is a graphical representation of the Ansible inventory groups and naming scheme used for `Site 1` in this example. This is replicated for `Site 2`.

![Ansible Groups](assets/images/ansible_groups_site1.svg)

## AVD Fabric Variables

To apply AVD variables to the nodes in the fabric, we make use of Ansible group_vars. How and where you define the variables is your choice. The group_vars table below is one example of AVD fabric variables for `Site 1`.

| group_vars/                | Description                                   |
| -------------------------- | --------------------------------------------- |
| SITE1_FABRIC.yml           | Fabric, Topology, and Device settings         |
| SITE1_SPINES.yml           | Device type for Spines                        |
| SITE1_LEAFS.yml            | Device type for Leafs                         |
| SITE1_NETWORK_SERVICES.yml | VLANs                                         |
| SITE1_NETWORK_PORTS.yml    | Port Profiles and Connected Endpoint settings |

Each group_vars file is listed in the following tabs.

=== "SITE1_FABRIC"
    At the Fabric level (SITE1_FABRIC), the following variables are defined in **group_vars/SITE1_FABRIC.yml**. The fabric name, design type (l2ls), node type defaults, interface links, and core interface P2P links are defined at this level. Other variables you must supply include:

    - spanning_tree_mode
    - spanning_tree_priority
    - mlag_peer_ipv4_pool

    The l3spine node will need these additonal variables set.

    - loopback_ipv4_pool
    - mlag_peer_l3_ipv4_pool
    - virtual_router_mac_address

    Variables applied under the node key type (l3spine/leaf) defaults section are inherited to nodes under each type. These variables may be overwritten under the node itself.

    The spine interface used by a particular leaf is defined from the leaf's perspective with a variable called `uplink_switch_interfaces`. For example, s1-leaf1 has a unique variable `uplink_switch_interfaces: [Ethernet2, Ethernet2]` defined. This means that s1-leaf1 is connected to `s1-spine1` Ethernet2 and `s1-spine2` Ethernet2, respectively.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_FABRIC.yml
    --8<--
    ```

=== "SITE1_SPINES"
    In an L2LS design, there are two types of spine nodes: `spine` and `l3spine`. In AVD. the node type defines the functionality and the EOS CLI configuration to be generated. For our L2LS topology, we will use node type `l3spine` to include SVI functionality.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_SPINES.yml
    --8<--
    ```

=== "SITE1_LEAFS"
    In an L2LS design, we have one type of leaf node: `leaf`. This will provide L2 functionality to the leaf nodes.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_LEAFS.yml
    --8<--
    ```

=== "SITE1_NETWORK_SERVICES"
    You add VLANs to the Fabric by updating the **group_vars/SITE1_NETWORK_SERVICES.yml**. Each VLAN will be given a name and a list of tags. The tags filter the VLAN to specific Leaf Pairs. These variables are applied to spine and leaf nodes since they are a part of this nested group.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_FABRIC_SERVICES.yml
    --8<--
    ```

=== "SITE1_NETWORK_PORTS"
    Our fabric would not be complete without connecting some devices to it. We define connected endpoints and port profiles in **group_vars/SITE1_NETWORKS_PORTS.yml**. Each endpoint adapter defines which switch port and port profile to use. In our lab, we have two hosts connected to the `site 1` fabric. The connected endpoints keys are used for logical separation and apply to interface descriptions. These variables are applied to spine and leaf nodes since they are a part of this nested inventory group.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_FABRIC_PORTS.yml
    --8<--
    ```

## Global Variables

In a multi-site environment, some variables need to be applied to all sites. The include: AAA, Local Users, NTP, Syslog, DNS, and TerminAttr. Instead of updating these same variables in multiple inventory group_vars, we can use a single global variable file and import the variables at playbook runtime. This allows us to make a single change that is applied to all sites. In our lab we use a global variable file `global_vars/global_dc-vars.yml`. In our playbooks we have a task that imports global variables before running other tasks.

### Task to import variables from a file

``` yaml
- name: Import Global Vars
    include_vars: "{{ item }}"
    with_items:
    - "../global_vars/global_dc_vars.yml"
```

???+ info

    The above task allows multiple files to be imported, using the `with_items` list.

### Example Global Vars File

??? eos-config annotate "global_vars/global_dc_vars.yml"
    ``` yaml
    ---
    # Credentials for CVP and EOS Switches
    ansible_user: arista
    ansible_password: "{{ lookup('env', 'LABPASSPHRASE') }}"
    ansible_network_os: arista.eos.eos
    # Configure privilege escalation
    ansible_become: true
    ansible_become_method: enable
    # HTTPAPI configuration
    ansible_connection: httpapi
    ansible_httpapi_port: 443
    ansible_httpapi_use_ssl: true
    ansible_httpapi_validate_certs: false
    ansible_python_interpreter: $(which python3)

    # Local Users
    local_users:
    arista:
        privilege: 15
        role: network-admin
        sha512_password: "{{ ansible_password | password_hash('sha512',  '65534') }}"

    # AAA
    aaa_authorization:
    exec:
        default: local

    # OOB Management network default gateway.
    mgmt_gateway: 192.168.0.1
    mgmt_interface: Management0
    mgmt_interface_vrf: default

    # NTP Servers IP or DNS name, first NTP server will be prefered, and sourced from Managment VRF
    ntp:
    servers:
        - name: 192.168.0.1
        iburst: true
        local_interface: Management0

    # Domain/DNS
    dns_domain: atd.lab

    # TerminAttr
    daemon_terminattr:
    # Address of the gRPC server on CloudVision
    # TCP 9910 is used on on-prem
    # TCP 443 is used on CV as a Service
    cvaddrs: # For single cluster
        - 192.168.0.5:9910
    # Authentication scheme used to connect to CloudVision
    cvauth:
        method: token
        token_file: "/tmp/token"
    # Exclude paths from Sysdb on the ingest side
    ingestexclude: /Sysdb/cell/1/agent,/Sysdb/cell/2/agent
    # Exclude paths from the shared memory table
    smashexcludes: ale,flexCounter,hardware,kni,pulse,strata

    # Point to Point Links MTU Override for Lab
    p2p_uplinks_mtu: 1500

    ```

## Data Models

AVD provides a network-wide data model and is typically broken into multiple group_vars file to simplify and categorize variables with their respective functions. We break the data model into three categories: topology, services, and ports.

### Fabric Topology

The physical fabric topology is defined be providing interface links between the spine and leaf nodes. The `group_vars/SITE1_FABRIC.yml` defines this portion of the data model. In our lab, the spines provide layer 3 routing of SVIs and P2P links by using a node type called `l3spines`. The leaf nodes are purely layer and use node type `leaf`. An AVD L2LS design type provides three node type keys: l3 spine, spine, and leaf. AVD Node Type documentation can be found [here](https://avd.sh/en/stable/roles/eos_designs/doc/node-types.html). Default node_type_keys for all design types are located [here](https://github.com/aristanetworks/ansible-avd/blob/devel/ansible_collections/arista/avd/roles/eos_designs/defaults/main/default-node-type-keys.yml).

#### Spine and Leaf Nodes

The spine and leaf nodes for each site is defined by the example data model below. Refer to the inline comments for variable definitions. Under each node_type_key you have key/value pairs for: defaults, node_groups, and nodes. Note that key/value pairs may be overwritten with the following descending order of precedence. The key/value closet to the node will be used.

<node_type_key>

1. defaults
2. node_groups
3. nodes

``` yaml
################################
# Spine Switches
################################

# node_type_key for providing L3 services
l3spine:
  # default key/values for all l3spine nodes
  defaults:
    # Platform dependent default settings for mlag timers, tcam profiles, LANZ, management interface
    platform: cEOS
    # Spanning Tree
    spanning_tree_mode: mstp
    spanning_tree_priority: 4096
    # Loopback0 pool, used in conjunction with value of `id:` under each node
    loopback_ipv4_pool: 10.1.252.0/24
    # IP Pool for MLAG peer link
    mlag_peer_ipv4_pool: 10.1.253.0/24
    # IP Pool for L3 peering over the MLAG peer link
    mlag_peer_l3_ipv4_pool: 10.1.254.0/24
    # Virtual MAC address used in vARP
    virtual_router_mac_address: 00:1c:73:00:dc:01
    # Default MLAG interfaces between spine nodes
    mlag_interfaces: [ Ethernet1, Ethernet6 ]
  # keyword for node groups, 2 nodes within a node group will form an MLAG pair
  node_groups:
    # User-defined node group name
    SPINES:
      # key word for nodes
      nodes:
        s1-spine1:
          # unique identifier used for IP address calculations
          id: 1
          # Management address assign to the defined management interface
          mgmt_ip: 192.168.0.10/24
        s1-spine2:
          id: 2
          mgmt_ip: 192.168.0.11/24

################################
# Leaf Switches
################################
leaf:
  defaults:
    platform: cEOS
    mlag_peer_ipv4_pool: 10.1.253.0/24
    spanning_tree_mode: mstp
    spanning_tree_priority: 16384
    # Default uplink switches from leaf perspective
    uplink_switches: [ s1-spine1, s1-spine2 ]
    # Default uplink interfaces on leaf nodes connecting to the spines
    uplink_interfaces: [ Ethernet2, Ethernet3 ]
    # Default leaf MLAG interfaces
    mlag_interfaces: [ Ethernet1, Ethernet6 ]
  node_groups:
    # User-defined node group name
    RACK1:
      # Filter which Vlans will be applied to the node_group, comma separated tags supported
      # Tags for each Vlan are defined in the SITE1_FABRIC_SERVICES.yml
      filter:
        tags: [ "10" ]
      nodes:
        s1-leaf1:
          id: 3
          mgmt_ip: 192.168.0.12/24
          # Define which interface is configured on the uplink switch
          # In this example s1-leaf1 connects to [ s1-spine1, s1-spine2 ]
          # on the following ports.  This will be unique to each leaf
          uplink_switch_interfaces: [ Ethernet2, Ethernet2 ]
        s1-leaf2:
          id: 4
          mgmt_ip: 192.168.0.13/24
          uplink_switch_interfaces: [ Ethernet3, Ethernet3 ]
    RACK2:
      filter:
        tags: [ "20" ]
      nodes:
        s1-leaf3:
          id: 5
          mgmt_ip: 192.168.0.14/24
          uplink_switch_interfaces: [ Ethernet4, Ethernet4 ]
        s1-leaf4:
          id: 6
          mgmt_ip: 192.168.0.15/24
          uplink_switch_interfaces: [ Ethernet5, Ethernet5 ]
```

#### Core Interfaces

Inside the same group_vars file, we define how each site is linked to the Core IP Network using point-to-point L3 links on the spine nodes. In our example, OSPF will be used to share routes between sites across the IP Network. The `core_interfaces` data model for `Site 1` follows.

``` yaml
core_interfaces:
  p2p_links:
    # s1-spine1 Ethernet7 to WANCORE
    - ip: [ 10.0.0.29/31, 10.0.0.28/31 ]
      nodes: [ s1-spine1, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    # s1-spine1 Ethernet8 to WANCORE
    - ip: [ 10.0.0.33/31, 10.0.0.32/31 ]
      nodes: [ s1-spine1, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true

    # s1-spine2 Ethernet7 to WANCORE
    - ip: [ 10.0.0.31/31, 10.0.0.30/31 ]
      nodes: [ s1-spine2, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    # s1-spine2 Ethernet8 to WANCORE
    - ip: [ 10.0.0.35/31, 10.0.0.34/31 ]
      nodes: [ s1-spine2, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true
```

The following diagram shows the P2P links from all four spine nodes to the IP Network. In our lab, the WAN IP Network are pre-configured with /31's running OSPF in area 0.0.0.0. We ran a playbook (`make preplab`) earlier to deploy these configurations. The core_interfaces for the spines in `Site 1` and `Site 2` are configured and deployed with AVD.

![Core Interfaces](assets/images/avd-core-interfaces.svg)

### Fabric Services

Fabric Services, such as Vlans, SVIs and VRFs are defined in this section. The following Site 1 example defines Vlans and SVIs for vlan `10` and `20` in the default VRF. Additional VRF definitions can also be applied.

``` yaml
---
tenants:
  # User-defined Tenant/Fabric name
  MY_FABRIC:
    # key-word
    vrfs:
      # Default VRF
      default:
        # key-word
        svis:
          # Vlan ID
          10:
            # Vlan Name
            name: 'Ten'
            # Tag assigned to Vlan. Used as a filter by each node_group
            tags: [ "10" ]
            enabled: true
            # SVI Virtual ARP address, used along with pre-defined virtual_router_mac_address
            ip_virtual_router_addresses:
              - 10.10.10.1
            # Which nodes to apply physical SVI address
            nodes:
              s1-spine1:
                ip_address: 10.10.10.2/24
              s1-spine2:
                ip_address: 10.10.10.3/24
          20:
            name: 'Twenty'
            tags: [ "20" ]
            enabled: true
            ip_virtual_router_addresses:
              - 10.20.20.1
            nodes:
              s1-spine1:
                ip_address: 10.20.20.2/24
              s1-spine2:
                ip_address: 10.20.20.3/24
```

### Fabric Ports

The Fabric needs to define ports for south bound interfaces toward connected endpoints such as: servers, appliances, firewalls, and other networking devices in the data center. This section makes use of port profiles and connected endpoints called `servers`. Documentation for [port_profiles](https://avd.sh/en/stable/roles/eos_designs/doc/connected-endpoints.html#port-profiles) and [connected endpoints](https://avd.sh/en/stable/roles/eos_designs/doc/connected-endpoints.html) are available to see all the options available.

The following data model defined two port profiles: PP-VLAN10 and PP-VLAN20. They define an access port profile for vlan `10` and `20`, respectively. In addition, two server endpoints (s1-host1 and s1-host2) are created to use these port profiles. There are optional and required fields. The optional fields are used for port descriptions in the EOS intended configurations.

``` yaml
port_profiles:

  PP-VLAN10:
    mode: "access"
    vlans: "10"
    spanning_tree_portfast: edge
  PP-VLAN20:
    mode: "access"
    vlans: "20"
    spanning_tree_portfast: edge

###########################################################
# ---------------- Endpoint Connectivity ---------------- #
###########################################################

servers:

# --------------------------------------------------------#
# Site1 RACK1 Endpoints
# --------------------------------------------------------#

  s1-host1:                                             # Server name
    rack: RACK1                                         # Informational RACK (optional)
    adapters:
      - endpoint_ports: [ eth1, eth2 ]                  # Server port to connect (optional)
        switch_ports: [ Ethernet4, Ethernet4 ]          # Switch port to connect server (required)
        switches: [ s1-leaf1, s1-leaf2 ]                # Switch to connect server (required)
        profile: PP-VLAN10                              # Port profile to apply (required)
        port_channel:
          mode: active

# --------------------------------------------------------#
# Site1 RACK2 Endpoints
# --------------------------------------------------------#

  s1-host2:                                             # Server name
    rack: RACK2                                         # Informational RACK (optional)
    adapters:
      - endpoint_ports: [ eth1, eth2 ]                  # Server port to connect (optional)
        switch_ports: [ Ethernet4, Ethernet4 ]          # Switch port to connect server (required)
        switches: [ s1-leaf3, s1-leaf4 ]                # Switch to connect server (required)
        profile: PP-VLAN20                              # Port profile to apply (required)
        port_channel:
          mode: active
```

## The Playbooks

There are two playbooks `build.yml` and `deploy.yml` used in our Lab. Expand the tabs below to reveal the content.

??? eos-config annotate "build.yml Playbook"
    ``` yaml
    --8<--
    workshops/assets/examples/avd/playbooks/build.yml
    --8<--
    ```

??? eos-config annotate "deploy.yml Playbook"
    ``` yaml
    --8<--
    workshops/assets/examples/avd/playbooks/deploy.yml
    --8<--
    ```

To make our lives easier, we use a `Makefile` to create aliases to run the playbooks and provide the needed options. This eliminates mistakes and typing long commands.

??? eos-config annotate "Makefile"
    ``` yaml
    --8<--
    workshops/assets/examples/avd/Makefile
    --8<--
    ```

For example, if we wanted to run a playbook to build configs for Site 1, we could enter the following command.

``` bash
ansible-playbook playbooks/build.yml -i sites/site_1/inventory.yml -e "target_hosts=SITE1_FABRIC"
```

Thankfully, there is a convenient way to simplify the above command with a Makefile entry like below.

``` bash
.PHONY: build-site-1
build-site-1: ## Build Configs
	ansible-playbook playbooks/build.yml -i sites/site_1/inventory.yml -e "target_hosts=SITE1_FABRIC"
```

Now, you can type the following to issue the same ansible-playbook command.

``` bash
make build-site-1
```

We will be using the following `make` commands several times in the upcoming lab. Review the above `Makefile` to see what each entry does. Try building some of your own entries.

Build configurations

```bash
# Build configs for Site 1
make build-site-1

# Build configs for Site 2
make build-site-2
```

Deploy configurations

```bash
# Deploy configs for Site 1
make deploy-site-1

# Deploy configs for Site 2
make deploy-site-2
```

## Next Steps

Continue on to the hand-ons lab. The lab guide is located **[here](avd-lab-guide.md)**.

[Continue to Lab Guide](avd-lab-guide.md){ .md-button .md-button--primary }
