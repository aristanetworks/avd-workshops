# AVD Lab Guide

## AVD Lab Guide Overview

The AVD Lab Guide is meant to be a follow along set of instructions to deploy a dual data center L2LS fabric design. The data model overview and details can be found [here](avd.md). In the following steps we will explore updating the data models to add services, ports and WAN links to our fabrics and test traffic between sites.

![Dual DC Topology](assets/images/dual-dc-topo.svg)

## Prepare Lab Environment

### **STEP #1** - Access the ATD Lab

Access the ATD Lab

### **STEP #2** - Fork and Clone branch to ATD Lab

Fork and Clone branch to ATD Lab

### **STEP #3** - Update AVD to latest version

Update AVD

### **STEP #4** - Setup Lab Password Environment Variable

- Set `LABPASSPHRASE` environment variable

### **STEP #5** - Prepare WAN IP Network and Test Hosts

- Prepare WAN/Hosts - `make preplab` playbook

## SITE 1 - Build and Deploy

### **STEP #1** - Build and Deploy Initial Fabric

Run & Deploy initial Build of Fabric (most features will be commented out)

- Basic configs will be created
- show configs and docs
- Deploy configs
- COMMIT  files changes before moving on

### **STEP #2** - Add Services to the Fabric

Add Services (vlan and svis)

- Uncomment services key/value pairs
- Run build & deploy
- Check file diffs
- Verify configs active on Site 1 devices (watch show mlag)
- COMMIT changes

### **STEP #3** - Add Ports for Hosts

Add Ports for Hosts

- Uncomment services key/value pairs
- Run build & deploy
- Check file diffs
- Verify configs active on Site 1 devices (watch show mlag)
- Verify pings are working between hosts within Site 1
- COMMIT changes

## SITE 2 - Build and Deploy

Repeat the previous steps for Site 2.

- Add Services
- Add Ports
- Build and Deploy Configs

At the end of this step, you should be able to ping between hosts within a site but not between sites. We will build connectivity to the `WAN IP Network` in the next section.

## Connect Sites to WAN IP Network

The WAN IP Network can be defined with the`core_interfaces` data model. Full data model documentation is located [here](https://avd.sh/en/stable/roles/eos_designs/doc/core-interfaces-BETA.html).

### Steps to be followed for Site 1 and 2

- Update data models for core_interfaces (Site 1 & 2)
  - Add `core_interfaces:` to the `SITE1_FABRIC.yml` and `SITE2_FABRIC.yml`
- Build & Deploy
- Validate Routing in Spines
- Test Traffic between hosts across sites

![Core Interfaces](assets/images/avd-core-interfaces.svg)

### Site #1

Add the following YAML code block to the bottom of `sites/site_1/group_vars/SITE1_FABRIC.yml`.

``` yaml
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

### Site #2

Add the following YAML code block to the bottom of `sites/site_1/group_vars/SITE2_FABRIC.yml`.

``` yaml
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

### Build and Deploy WAN IP Network connectivity

``` bash
make build-site-1
make build-site-2
make deploy-site-1
make deploy-site-2
```

### Test traffic between sites

From `s1-host1` ping both `s2-host1` & `s1-host2`

``` bash
# s2-host1
ping 10.30.30.100

# s2-host2
ping 10.40.40.100
```

## **Congratulations!**

You have just built a multi-site L2LS network without touching the Cli on a single switch.
