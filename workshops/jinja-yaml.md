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
# These are some examples of scalars

integer: 10
boolean: true
string: "Welcome to the Automation Workshop"
```

<br>

Sometimes, a scalar is referred to as only the initial part of the line of text such as a list of names or locations like such.

```yaml
# Another scalar example

- White Rim Trail
- Hells Revenge
- Long Canyon Road
- Black Bears Pass
- Ophir Pass
```

<br>

In normal day to day use, you will see scalars referred to as **key value pair mappings**, or just **mappings** for short.  A mapping is the data label, followed by its value, separated by a **`:`** colon.

```yaml
# Some Non-Specific Mappings
trip_start: "August 12"
trip_end: "September 5"
first_stop: "Duck Creek Village, UT"
first_trail: "Fins and Things"
stuck_counter: 1
```

Here are some device specific mappings.

```yml
#

### **Boolean**

### **Lists**

### **Dictionary or Dicts**

### **Nested Data Structures**

## **YAML File Examples**

### **Generic Example**

### **Network Config Example**

<br>
<br>
<br>

# **Jinja**

## **What is Jinja?**

## **What is Jinja Used For?**

## **Jinja Syntax**

### **Variable Substitution**

### **Conditionals**

### **Loops**

### **Filters**

## **Jinja Templates**

### **Generic Template Example**

### **Network Config Template Example**

<br>
<br>
<br>

# **The Jinja YAML Relationship**

<br>
<br>
<br>

# **Final Output - Tying It All Together**
