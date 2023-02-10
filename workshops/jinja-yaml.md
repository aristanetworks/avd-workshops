![Jinja YAML Banner](assets/images/jinja-yaml-banner.png)

# Welcome to YAML & Jinja

This section will cover both Jinja and YAML, which are two interdependent pieces of basic configuration automation framework.  While both YAML and Jinja can get relatively complex with what they can accomplish and what can be done with them, we will only cover what is necessary to utilize these tools for network automation and DevOps.

# YAML

## What is YAML?

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

## What is YAML Used For?

## Common YAML Syntax

### Comments

### Mappings

### Boolean

### Lists

### Dictionary or Dicts

### Nested Data Structures

## YAML File Examples

### Generic Example

### Network Config Example

<br>
<br>
<br>

# Jinja

## What is Jinja?

## What is Jinja Used For?

## Jinja Syntax

### Variable Substitution

### Conditionals

### Loops

### Filters

## Jinja Templates

### Generic Template Example

### Network Config Template Example

<br>
<br>
<br>

# The Jinja YAML Relationship

<br>
<br>
<br>

# Final Output - Tying It
