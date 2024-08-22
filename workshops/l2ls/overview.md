# Arista CI Workshop

This workshop will leverage open-source tools to build a network CI pipeline for configuration development, deployment, documentation, and validation. In addition, the open-source tooling enables us to manage our network environment as code.

This section will cover the following:

- Arista Validated Designs (AVD) Ansible Collection
- Network Data Models
- Initial Deployment (Day 0 Provisioning)
- Ongoing Operations (Day 2 and Beyond)
- Validation and Troubleshooting

Each attendee will receive a dedicated virtual lab environment with Git, VS Code, and Ansible installed and ready to use.

Attendees will need the following:

- A laptop
- An account on [GitHub](https://github.com/)
- Familiarity with the concepts and tools covered in the previous [Automation Fundamentals workshop](https://aristanetworks.github.io/avd-workshops/) (Git, VS Code, Jinja/YAML, Ansible)

## Lab Topology Overview

Throughout this section, we will use the following dual data center topology. Click on the image to zoom in for details.

![Dual DC Topology](../assets/images/dual-dc-topo.svg)

## Basic EOS Switch Configuration

Basic connectivity between the Ansible controller host and the switches must be established before Ansible can be used to deploy configurations. The following should be configured on all switches:

- Switch Hostname
- IP enabled interface
- Username and Password defined
- Management eAPI enabled

???+ info

    In the ATD environment, cEOS virtual switches use `Management0` in the default VRF. When using actual hardware or vEOS switches, `Management1` is used. The included basic switch configurations may need to be adjusted for your environment.

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

![Ansible Groups](../assets/images/ansible_groups_site1.svg)

## AVD Fabric Variables

To apply AVD variables to the nodes in the fabric, we make use of Ansible group_vars. How and where you define the variables is your choice. The group_vars table below is one example of AVD fabric variables for `Site 1`.

| group_vars/                | Description                                   |
| -------------------------- | --------------------------------------------- |
| SITE1_FABRIC.yml           | Fabric, Topology, and Device settings         |
| SITE1_SPINES.yml           | Device type for Spines                        |
| SITE1_LEAFS.yml            | Device type for Leafs                         |
| SITE1_NETWORK_SERVICES.yml | VLANs, VRFs, SVIs                             |
| SITE1_NETWORK_PORTS.yml    | Port Profiles and Connected Endpoint settings |

Each group_vars file is listed in the following tabs.

=== "SITE1_FABRIC"
    At the Fabric level (SITE1_FABRIC), the following variables are defined in **group_vars/SITE1_FABRIC.yml**. The fabric name, design type (l2ls), node type defaults, interface links, and core interface P2P links are defined at this level. Other variables you must supply include:

    - spanning_tree_mode
    - spanning_tree_priority
    - mlag_peer_ipv4_pool

    The l3spine node will need these additional variables set.

    - loopback_ipv4_pool
    - mlag_peer_l3_ipv4_pool
    - virtual_router_mac_address

    Variables applied under the node key type (l3spine/leaf) defaults section are inherited by nodes under each type. These variables may be overwritten under the node itself.

    The spine interface used by a particular leaf is defined from the leaf's perspective with a variable called `uplink_switch_interfaces`. For example, s1-leaf1 has a unique variable `uplink_switch_interfaces: [Ethernet2, Ethernet2]` defined. This means that s1-leaf1 is connected to `s1-spine1` Ethernet2 and `s1-spine2` Ethernet2, respectively.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_FABRIC.yml
    --8<--
    ```

=== "SITE1_SPINES"
    In an L2LS design, there are two types of spine nodes: `spine` and `l3spine`. In AVD, the node type defines the functionality and the EOS CLI configuration to be generated. For our L2LS topology, we will use node type `l3spine` to include SVI functionality.

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
    Our fabric would only be complete by connecting some devices to it. We define connected endpoints and port profiles in **group_vars/SITE1_NETWORKS_PORTS.yml**. Each endpoint adapter defines which switch port and port profile to use. Our lab has two hosts connected to the `site 1` fabric. The connected endpoints keys are used for logical separation and apply to interface descriptions. These variables are applied to the spine and leaf nodes since they are a part of this nested inventory group.

    ``` yaml
    --8<--
    workshops/assets/examples/avd/site_1/group_vars/SITE1_FABRIC_PORTS.yml
    --8<--
    ```

## Global Variables

In a multi-site environment, some variables must be applied to all sites. They include AAA, Local Users, NTP, Syslog, DNS, and TerminAttr. Instead of updating these same variables in multiple inventory group_vars, we can use a single global variable file and import the variables at playbook runtime. This allows us to make a single change applied to all sites.

For example, in our lab, we use a global variable file `global_vars/global_dc-vars.yml`.

AVD provides a [`global_vars`](https://avd.arista.com/4.10/docs/plugins/Vars_plugins/global_vars.html) plugin that enables the use of global variables.

The `global_vars` plugin must be enabled in the `ansible.cfg` file as shown below:

``` yaml
#enable global vars
vars_plugins_enabled = arista.avd.global_vars, host_group_vars

#define global vars path
[vars_global_vars]
paths = ../../global_vars
```

???+ info

    If a folder is used as in the example above, all files in the folder will be parsed in alphabetical order.

### Example Global Vars File

??? eos-config annotate "global_vars/global_dc_vars.yml"
    ``` yaml
    --8<--
    workshops/assets/examples/avd/global_dc_vars.yml
    --8<--
    ```

## Data Models

AVD provides a network-wide data model and is typically broken into multiple group_vars files to simplify and categorize variables with their respective functions. We break the data model into three categories: topology, services, and ports.

### Fabric Topology

The physical fabric topology is defined by providing interface links between the spine and leaf nodes. The `group_vars/SITE1_FABRIC.yml` file defines this portion of the data model. In our lab, the spines provide layer 3 routing of SVIs and P2P links using a node type called `l3spines`. The leaf nodes are purely layer 2 and use node type `leaf`. An AVD L2LS design type provides three node type keys: l3 spine, spine, and leaf. AVD Node Type documentation can be found **[here](https://avd.arista.com/4.10/roles/eos_designs/docs/input-variables.html#node-type-variables)**.

#### Spine and Leaf Nodes

The example data model below defines each site's spine and leaf nodes for each site. Refer to the inline comments for variable definitions. Under each node_type_key you have key/value pairs for defaults, node_groups, and nodes. Note that key/value pairs may be overwritten with the following descending order of precedence. The key/value closest to the node will be used.

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
    mlag_peer_ipv4_pool: 10.1.253.0/31
    # IP Pool for L3 peering over the MLAG peer link
    mlag_peer_l3_ipv4_pool: 10.1.253.2/31
    # Virtual MAC address used in vARP
    virtual_router_mac_address: 00:1c:73:00:dc:01
    # Default MLAG interfaces between spine nodes
    mlag_interfaces: [ Ethernet1, Ethernet6 ]
  # keyword for node groups, two nodes within a node group will form an MLAG pair
  node_groups:
    # User-defined node group name
    - group: SPINES
      # key word for nodes
      nodes:
        - name: s1-spine1
          # unique identifier used for IP address calculations
          id: 1
          # Management address assigned to the defined management interface
          mgmt_ip: 192.168.0.10/24
        - name: s1-spine2
          id: 2
          mgmt_ip: 192.168.0.11/24

################################
# Leaf Switches
################################
leaf:
  defaults:
    platform: cEOS
    mlag_peer_ipv4_pool: 10.1.253.0/31
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
    - group: RACK1
      # Filter which Vlans will be applied to the node_group, comma-separated tags supported
      # Tags for each Vlan are defined in the SITE1_FABRIC_SERVICES.yml
      filter:
        tags: [ "Web" ]
      nodes:
        - name: s1-leaf1
          id: 1
          mgmt_ip: 192.168.0.12/24
          # Define which interface is configured on the uplink switch
          # In this example s1-leaf1 connects to [ s1-spine1, s1-spine2 ]
          # on the following ports. This will be unique to each leaf
          uplink_switch_interfaces: [ Ethernet2, Ethernet2 ]
        - name: s1-leaf2
          id: 2
          mgmt_ip: 192.168.0.13/24
          uplink_switch_interfaces: [ Ethernet3, Ethernet3 ]
    - group: RACK2
      filter:
        tags: [ "App" ]
      nodes:
        - name: s1-leaf3
          id: 3
          mgmt_ip: 192.168.0.14/24
          uplink_switch_interfaces: [ Ethernet4, Ethernet4 ]
        - name: s1-leaf4
          id: 4
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

The following diagram shows the P2P links to the IP Network from all four spine nodes. The WAN IP Network is pre-configured in our lab with /31's running OSPF in area 0.0.0.0. The core_interfaces for the spines in `Site 1` and `Site 2` are configured and deployed with AVD.

![Core Interfaces](../assets/images/avd-core-interfaces.svg)

### Fabric Services

Fabric Services, such as VLANs, SVIs, and VRFs, are defined in this section. The following Site 1 example defines VLANs and SVIs for VLANs `10` and `20` in the default VRF. Additional VRF definitions can also be applied.

``` yaml
---
tenants:
  # User-defined Tenant/Fabric name
  - name: MY_FABRIC
    # key-word
    vrfs:
      # Default VRF
      - name: default
        # key-word
        svis:
          # Vlan ID
          - id: 10
            # Vlan Name
            name: 'Ten'
            # Tag assigned to Vlan. Used as a filter by each node_group
            tags: [ "Web" ]
            enabled: true
            # SVI Virtual ARP address, used along with pre-defined virtual_router_mac_address
            ip_virtual_router_addresses:
              - 10.10.10.1
            # Which nodes to apply physical SVI address
            nodes:
              - node: s1-spine1
                ip_address: 10.10.10.2/24
              - node: s1-spine2
                ip_address: 10.10.10.3/24
          - id: 20
            name: 'Twenty'
            tags: [ "App" ]
            enabled: true
            ip_virtual_router_addresses:
              - 10.20.20.1
            nodes:
              - node: s1-spine1
                ip_address: 10.20.20.2/24
              - node: s1-spine2
                ip_address: 10.20.20.3/24
```

### Fabric Ports

The Fabric must define ports for southbound interfaces toward connected endpoints such as servers, appliances, firewalls, and other networking devices in the data center. This section uses port profiles and connected endpoints called `servers`. Documentation for [port_profiles](https://avd.sh/en/v4.1.0/roles/eos_designs/docs/input-variables.html#port-profiles-settings) and [connected endpoints](https://avd.sh/en/v4.1.0/roles/eos_designs/docs/input-variables.html#endpoint-connectivity) are available to see all the options available.

The following data model defined two port profiles: PP-VLAN10 and PP-VLAN20. They define an access port profile for VLAN `10` and `20`, respectively. In addition, two server endpoints (s1-host1 and s1-host2) are created to use these port profiles. There are optional and required fields. The optional fields are used for port descriptions in the EOS intended configurations.

``` yaml
---
port_profiles:

  - profile: PP-VLAN10
    mode: "access"
    vlans: "10"
    spanning_tree_portfast: edge
  - profile: PP-VLAN20
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

  - name: s1-host1                                      # Server name
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

  - name: s1-host2                                      # Server name
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

Two playbooks, `build.yml` and `deploy.yml` are used in our lab. Expand the tabs below to reveal the content.

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

Thankfully, a convenient way to simplify the above command is with a Makefile entry like the one below.

``` bash
.PHONY: build-site-1
build-site-1: ## Build Configs
  ansible-playbook playbooks/build.yml -i sites/site_1/inventory.yml -e "target_hosts=SITE1_FABRIC"
```

Now, you can type the following to issue the same ansible-playbook command.

``` bash
make build-site-1
```

In the upcoming lab, we will use the following `make` commands several times. First, review the above `Makefile` to see what each entry does. Then, try building some custom entries.

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

[Continue to Lab Guide](l2ls-lab-guide.md){ .md-button .md-button--primary }
