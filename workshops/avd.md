# Arista CI Workshop

This section we will leverage open source tools to build a network CI pipeline for configuration development, deployment, documentation and validation. Enabling us to manage our network environment as code.

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
workshops/example_files/s1-spine1.cfg
--8<--
```

## Ansible Inventory

Our lab L2LS topology contains two sites, `Site 1` and `Site 2`. We need to create the Ansible inventory for each site. The following is a graphical representation of the Ansible inventory groups and naming scheme used for `Site 1` in this example.  This is replicated for `Site 2`.

![Ansible Groups](assets/images/ansible_groups_site1.svg)

We have created two separate directories for each sites under the `sites` sub-directory in our repo.

``` text
├── sites/
  ├── site_1/
  ├── site_2/
```

## AVD Fabric Variables

## The Playbooks

## EOS Intended Configurations
