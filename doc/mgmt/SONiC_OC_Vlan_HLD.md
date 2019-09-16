# VLAN
Layer 2 Forwarding Enhancements.
# High Level Design Document
#### Rev 0.1

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)

# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev |     Date    |       Author       | Change Description                |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 09/05/2019  |   Justine Jose      | Initial version                   |

# About this Manual
This document provides general information about VLAN configuration in SONiC using the management framework.

# Scope
Covers Northbound interface for the VLAN feature, as well as Unit Test cases.

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**                 | **Meaning**                         |
|--------------------------|-------------------------------------|
| VLAN                      | Virtual Local Area Network         |

# 1 Feature Overview

Provide management framework capabilities to handle:
- VLAN creation and deletion
- Addition of tagged/un-tagged ports to vlan
- Removal of tagged/un-tagged ports from vlan
- Associated show commands.

## 1.1 Requirements

### 1.1.1 Functional Requirements

Provide management framework support to existing SONiC capabilities with respect to VLANs.

### 1.1.2 Configuration and Management Requirements
- IS-CLI style configuration and show commands
- REST API support
- gNMI Support

Details described in Section 3.

## 1.2 Design Overview

### 1.2.1 Basic Approach
Need a description here

### 1.2.2 Container
Management container

### 1.2.3 SAI Overview
N/A

# 2 Functionality
## 2.1 Target Deployment Use Cases
Wordy description, with diagrams if possible
## 2.2 Functional Description
Wordy description

# 3 Design
## 3.1 Overview
big picture view of the actors involved.

## 3.2 User Interface
### 3.2.1 Data Models
**openconfig-vlan.yang** (Config for Set Op and State for Get Op)

- ``/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/[config|state]/[interface-mode/access-vlan/trunk-vlans]``

Exception - "native-vlan" is not supported in the above config and state containers.

**openconfig-interfaces.yang** (Get Support)

- ``/openconfig-interfaces:interfaces/ interface={name}/state/[admin-status|mtu]``

Note - SONiC doesn't support other attributes for VLAN Interface.


### 3.2.2 CLI

#### 3.2.2.1 Configuration Commands
````
VLAN Creation:-
sonic(config)# interface vlan <vlan-id>

VLAN Deletion:-
sonic(config)# no interface vlan <vlan-id>

Trunk VLAN addition to Member Port:-
sonic(conf-if-Ethernet4)# switchport trunk allowed vlan <vlan-id>

Trunk VLAN removal from Member Port:-
sonic(conf-if-Ethernet4)# no switchport trunk allowed vlan <vlan-id>

Access VLAN addition to Member Port:-
sonic(conf-if-Ethernet4)# switchport access vlan <vlan-id>

Access VLAN removal from Member Port:-
sonic(conf-if-Ethernet4)# no switchport access vlan
````
#### 3.2.2.2 Show Commands
````
sonic# show vlan
Q: A - Access (Untagged), T - Tagged
    NUM    Q Ports
    5      T Ethernet24
    10
    20     A Ethernet4

sonic# show vlan <vlan-id>
Q: A - Access (Untagged), T - Tagged
    NUM    Q Ports
    5      T Ethernet24
           A Ethernet20
````
#### 3.2.2.3 Debug Commands
N/A

#### 3.2.2.4 IS-CLI Compliance
N/A

### 3.2.3 REST API Support

**PATCH**
- `/openconfig-interfaces:interfaces/ interface={name}`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans | interface-mode]`

Note: Expects Interface mode to be passed for ACCESS/TRUNK vlan configs.

**DELETE**
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/config/[access-vlan | trunk-vlans]`
- `/openconfig-interfaces:interfaces/ interface={name}`

**GET**
- `/openconfig-interfaces:interfaces/ interface={name}/state/[admin-status | mtu]`
- `/openconfig-interfaces:interfaces/interface={name}/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan/State/[access-vlan | trunk-vlans]`

# 4 Flow Diagrams
N/A

# 5 Error Handling
TBD

# 6 Serviceability and Debug
TBD

# 7 Warm Boot Support
N/A

# 8 Scalability
N/A

# 9 Unit Test
- Create VLAN A, verify it using #show vlan command and SONiC CLI commands.
- Add an untagged-port to VLAN A, verify it using #show vlan <id> command and SONiC CLI commands.
- Add 2 tagged-ports to VLAN B, verify it using #show vlan command and SONiC CLI commands.
- Remove un-tagged port from VLAN A, verify it using #show vlan <id> command and SONiC CLI commands.
- Remove all the tagged-ports from VLAN B, verify it using #show vlan <id> command and SONiC CLI commands.
- Delete VLAN, verify it using #show vlan command and SONiC CLI commands.

# 10 Internal Design Information
Internal BRCM information to be removed before sharing with the community.