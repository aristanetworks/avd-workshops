# AVD Lab Guide

## AVD Lab Guide Overview

The AVD Lab Guide is a follow-along set of instructions to deploy a dual data center L2LS fabric design. The data model overview and details can be found [here](avd.md). In the following steps, we will explore updating the data models to add services, ports, and WAN links to our fabrics and test traffic between sites.

In this example, the ATD lab is used to create the L2LS Dual Data Center topology below. The IP Network cloud (orange area) is pre-provisioned and is comprised of the border and core nodes in the ATD topology. Our focus will be creating the L2LS AVD data models to build and deploy configurations for Site 1 and Site 2 (blue areas) and connect them to the IP Network.

![Dual DC Topology](assets/images/topologies.svg)

### Host Addresses

| Host     |  IP Address  |
|:--------:|:------------:|
| s1-host1 | 10.10.10.100 |
| s1-host2 | 10.20.20.100 |
| s2-host1 | 10.30.30.100 |
| s2-host2 | 10.40.40.100 |

## **Prepare Lab Environment**

### STEP #1 - Access the ATD Lab and configure global Git settings

Connect to your ATD Lab and start the Programmability IDE. Next, create a new Terminal.

``` bash
cd /home/coder/project/labfiles/workshops-avd
git config --global user.name "FirstName LastName"
git config --global user.email "name@example.com"
```

### STEP #2 - Fork and Clone branch to ATD Lab

An ATD Dual Data Center L2LS data model is posted on GitHub here: [https://github.com/PacketAnglers/workshops-avd](https://github.com/PacketAnglers/workshops-avd)

- Fork this [repository](https://github.com/PacketAnglers/workshops-avd) to your own GitHub account.
- Next, clone your forked repo to your ATD lab instance.

``` bash
cd /home/coder/project/labfiles
git clone <your copied URL>
cd workshops-avd
```

### STEP #3 - Update AVD to the latest version

AVD has been pre-installed in your lab environment. However, it is on an older version. The following steps will update AVD and modules to the latest versions.

``` bash
ansible-galaxy collection install arista.avd arista.cvp --force
export ARISTA_AVD_DIR=$(ansible-galaxy collection list arista.avd --format yaml | head -1 | cut -d: -f1)
pip3 config set global.disable-pip-version-check true
pip3 install -r ${ARISTA_AVD_DIR}/arista/avd/requirements.txt
```

???+ Warning "IMPORTANT"

    You must run these commands when you start your lab or a new shell (terminal).

### STEP #4 - Setup Lab Password Environment Variable

Each lab comes with a unique password. We set an environment variable called `LABPASSPHRASE` with the following command. The variable is later used to generate local user passwords and connect to our switches to push configs.

``` bash
export LABPASSPHRASE=`cat /home/coder/.config/code-server/config.yaml| grep "password:" | awk '{print $2}'`
```

You can view the password is set. This is the same password displayed when you click the link to access your lab.

``` bash
echo $LABPASSPHRASE
```

???+ Warning "IMPORTANT"

    You must run this step when you start your lab or a new shell (terminal).

### STEP #5 - Prepare WAN IP Network and Test Hosts

The last step in preparing your lab is to push pre-defined configurations to the WAN IP Network (cloud) and the four hosts used to test traffic. The spines from each site will connect to the WAN IP Network with P2P links. The hosts (two per site) have port-channels to the leaf pairs and are pre-configured with an IP address and route to reach the other hosts.

Run the following to push the configs.

``` bash
make preplab
```

## **Build and Deploy Dual Data Center L2LS Network**

This section will review and update the existing L2LS data model. We will add features to enable VLANs, SVIs, connected endpoints, and P2P links to the WAN IP Network. After the lab, you will have enabled an L2LS dual data center network through automation with AVD. YAML data models and Ansible playbooks will be used to generate EOS CLI configurations and deploy them to each site. We will start by focusing on building out Site 1 and then repeat similar steps for Site 2. Finally, we will enable connectivity to the WAN IP Network to allow traffic to pass between sites.

### **Summary of Steps**

1. Build and Deploy `Site 1`
2. Build and Deploy `Site 2`
3. Connect sites to WAN IP Network
4. Verify routing
5. Test traffic

## **Site 1**

### STEP #1 - Build and Deploy Initial Fabric

The initial fabric data model key/value pairs have been pre-populated in the following group_vars files in the `sites/site_1/group_vars/` directory.

- SITE1_FABRIC_PORTS.yml
- SITE1_FABRIC_SERVICES.yml
- SITE1_FABRIC.yml
- SITE1_LEAFS.yml
- SITE1_SPINES.yml

Review these files to understand how they relate to the topology above.

At this point, we can build and deploy our initial configurations to the topology.

``` bash
make build-site-1
```

AVD creates a separate markdown and EOS configuration file per switch. In addition, you can review the files in the `documentation` and `intended` folders per site.

![Docs and Configs](assets/images/docs-configs.png){: style="width:300px"}

Now, deploy the configurations to Site 1 switches.

``` bash
make deploy-site-1
```

Login to your switches to verify the current configs (`show run`) match the ones created in `intended/configs` folder.

You can also check the current state for MLAG, VLANs, interface, and port-channels.

``` bash
show mlag
```

``` bash
show vlan brief
```

``` bash
show ip interface brief
```

``` bash
show port-channel
```

The basic fabric with MLAG peers and port-channels between leaf and spines are now created. Next up, we will add VLAN and SVI services to the fabric.

### STEP #2 - Add Services to the Fabric

The next step is to add Vlans and SVIs to the fabric. The services data model file `SITE1_FABRIC_SERVICES.yml` is pre-populated with Vlans and SVIs `10` and `20` in the default VRF.

Open `SITE1_FABRIC_SERVICES.yml` and uncomment lines 1-28, then run the build & deploy process again.

???+ tip
    :writing_hand: In VS Code, you can toggle comments on/off by selecting the text and pressing ***windows*** ++ctrl++ + ++slash++ or ***mac*** ++cmd++ + ++slash++.

``` bash
make build-site-1
```

``` bash
make deploy-site-1
```

Log into `s1-spine1` and `s1-spine2` and verify the SVIs `10` and `20` exist.

``` bash
show ip interface brief
```

It should look similar to the following:

``` text
                                           Address
Interface         IP Address            Status       Protocol            MTU    Owner
----------------- --------------------- ------------ -------------- ----------- -------
Loopback0         10.1.252.1/32         up           up                65535
Management0       192.168.0.10/24       up           up                 1500
Vlan10            10.10.10.2/24         up           up                 1500
Vlan20            10.20.20.2/24         up           up                 1500
Vlan4093          10.1.254.0/31         up           up                 1500
Vlan4094          10.1.253.0/31         up           up                 1500
```

You can verify the recent configuration session was created.

???+ info
    When configuration is applied via a configuration session, EOS will create a "checkpoint" of the configuration. This checkpoint is a snapshot
    of the configuration as it was **prior** to the configuration session being committed.

``` bash
show clock
```

``` bash
show configuration sessions detail
```

List the recent checkpoints.

``` bash
show config checkpoints
```

View the contents of the latest checkpoint file.

``` bash
more checkpoint:< filename >
```

See the difference between the running config and the latest checkpoint file.

???+ tip
    This will show the differences between the current device configuration
    and what the configuration was before we did our `make deploy` command.

``` bash
diff checkpoint:< filename > running-config
```

### STEP #3 - Add Ports for Hosts

Let's configure port-channels to our hosts (`s1-host1` and `s1-host2`).

Open `SITE1_FABRIC_PORTS.yml` and uncomment lines 16-44, then run the build & deploy process again.

``` bash
make build-site-1
```

``` bash
make deploy-site-1
```

At this point, hosts should be able to ping each other across the fabric.

From `s1-host1`, run a ping to `s1-host2`.

``` bash
ping 10.20.20.100
```

``` text
PING 10.20.20.100 (10.20.20.100) 72(100) bytes of data.
80 bytes from 10.20.20.100: icmp_seq=1 ttl=63 time=30.2 ms
80 bytes from 10.20.20.100: icmp_seq=2 ttl=63 time=29.5 ms
80 bytes from 10.20.20.100: icmp_seq=3 ttl=63 time=28.8 ms
80 bytes from 10.20.20.100: icmp_seq=4 ttl=63 time=24.8 ms
80 bytes from 10.20.20.100: icmp_seq=5 ttl=63 time=26.2 ms
```

Site 1 fabric is now complete.

## **Site 2**

Repeat the previous three steps for Site 2.

- Add Services
- Add Ports
- Build and Deploy Configs
- Verify ping traffic between hosts `s2-host1` and `s2-host2`

At this point, you should be able to ping between hosts within a site but not between sites. For this, we need to build connectivity to the `WAN IP Network`. This is covered in the next section.

## **Connect Sites to WAN IP Network**

The WAN IP Network is defined by the `core_interfaces` data model. Full data model documentation is located [here](https://avd.sh/en/stable/roles/eos_designs/doc/core-interfaces-BETA.html).

The data model defines P2P links (`/31s`) on the spines with a stanza per link. See details in the graphic below. Each spine has two links to the WAN IP Network configured on ports `Ethernet7` and `Ethernet8`. OSPF is added to these links as well.

![Core Interfaces](assets/images/avd-core-interfaces.svg)

### **Add P2P Links to WAN IP Network for Site 1 and 2**

Add each site's `core_interfaces` dictionary (shown below) to the bottom of the following files `SITE1_FABRIC.yml` and `SITE2_FABRIC.yml`

### **Site #1**

Add the following code block to the bottom of `sites/site_1/group_vars/SITE1_FABRIC.yml`.

``` yaml
##################################################################
# WAN/Core Edge Links
##################################################################

core_interfaces:
  p2p_links:

    - ip: [ 10.0.0.29/31, 10.0.0.28/31 ]
      nodes: [ s1-spine1, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.33/31, 10.0.0.32/31 ]
      nodes: [ s1-spine1, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.31/31, 10.0.0.30/31 ]
      nodes: [ s1-spine2, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.35/31, 10.0.0.34/31 ]
      nodes: [ s1-spine2, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true
```

### **Site #2**

Add the following code block to the bottom of `sites/site_1/group_vars/SITE2_FABRIC.yml`.

``` yaml
##################################################################
# WAN/Core Edge Links
##################################################################

core_interfaces:
  p2p_links:

    - ip: [ 10.0.0.37/31, 10.0.0.36/31 ]
      nodes: [ s2-spine1, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.41/31, 10.0.0.40/31 ]
      nodes: [ s2-spine1, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.39/31, 10.0.0.38/31 ]
      nodes: [ s2-spine2, WANCORE ]
      interfaces: [ Ethernet7, Ethernet2 ]
      include_in_underlay_protocol: true

    - ip: [ 10.0.0.43/31, 10.0.0.42/31 ]
      nodes: [ s2-spine2, WANCORE ]
      interfaces: [ Ethernet8, Ethernet2 ]
      include_in_underlay_protocol: true
```

### **Build and Deploy WAN IP Network connectivity**

``` bash
make build-site-1 build-site-2 deploy-site-1 deploy-site-2
```

???+ tip
    Daisy chaining "Makesies" is a great way to run a series of tasks with a single CLI command :grinning:

### **Check routes on spine nodes**

From the spines, verify that they can see routes to the following networks where the hosts reside.

- 10.10.10.0/24
- 10.20.20.0/24
- 10.30.30.0/24
- 10.40.40.0/24

``` bash
show ip route
```

### **Test traffic between sites**

From `s1-host1` ping both `s2-host1` & `s1-host2`

``` bash
# s2-host1
ping 10.30.30.100
```

``` bash
# s2-host2
ping 10.40.40.100
```

## **Congratulations!**

You have built a multi-site L2LS network without touching the CLI on a single switch.
