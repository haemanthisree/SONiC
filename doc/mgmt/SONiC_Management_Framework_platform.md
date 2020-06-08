# Feature Name
Platform commands in Management Framework
# High Level Design Document
#### Rev 0.3

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)

# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev |     Date    |       Author       | Change Description                                                     |
|:---:|:-----------:|:------------------:|------------------------------------------------------------------------|
| 0.1 | 06/08/2020  |   Garrick He       | Initial version                                                        |


# About this Manual
This document provides general information about platform command support in SONiC Management Framework
# Scope
This document describes the high level design of platform command support in SONiC Management Framework.


# 1 Feature Overview
This feature will allow the user to get various platform information through KLISH CLI or REST/gRPC GET. Information available are:

* System sensors - Currently this is done through a dbus-query instead of state Db.
* System EEPROM
* PSU information
* Fan information
* Temperature sensor information

## 1.1 Requirements


### 1.1.1 Functional Requirements

### 1.1.2 Configuration and Management Requirements
1. CLI show support
2. REST GET support
3. gNMI GET support

### 1.1.3 Scalability Requirements

### 1.1.4 Warm Boot Requirements

## 1.2 Design Overview
### 1.2.1 Basic Approach
1. Implement platform support using transformer in sonic-mgmt-framework.

### 1.2.2 Container
There will be changes in the sonic-mgmt-framework container. The backend will be modifications to translib done through Transformer. YANG and Transformer changes are done through:

1. openconfig-platform YANG model. Changes will be done through extensions in the openconfig-platform-ext.yang file
2. openconfig-platform-annot.yang - annotations for the openconfig.yang
3. xfmr_platform.go - Transformer functions.

There will be additional files added to:
1. XML file for the CLI
2. Python script to handle CLI request (actioner)
3. Jinja template to render CLI output (renderer)


### 1.2.3 SAI Overview


# 2 Functionality
## 2.1 Target Deployment Use Cases

## 2.2 Functional Description


# 3 Design
## 3.1 Overview
## 3.2 DB Changes
There will be no dB changes (writes) for this feature, there will only be dB lookups (reads).
### 3.2.2 APP DB
### 3.2.3 STATE DB
There are certain requirements for the dB keys for PSU, fan, and temperature components.

For PSU, the table and key scheme is as follows:

`PSU_INFO | PSU #`

Example: `PSU INFO | PSU 1`

for fan, the table and key schema is:

`FAN_INFO | FAN #` **for system fans or**

`FAN_INFO | PSU # FAN #` **for PSU fans**

Example: `FAN INFO | FAN 1`

`FAN INFO | PSU 1 FAN 1`

For temperature sensor, the table and key schema is:

`TEMPERATURE_INFO|TEMP #`

Example: `TEMPERATURE_INFO|TEMP 1`

The other requirement for these three components is their key **MUST** be allocated consectively for each component category.
For example, if you have 5 fans on the system, their keys must be {FAN 1, FAN 2, FAN 3, FAN 4, FAN 5}. They **cannot** be non-consective,
{FAN 1, FAN 5, FAN 7, FAN 8, FAN 9}. **ALL KEYS ARE 1-INDEXED**
### 3.2.4 ASIC DB
### 3.2.5 COUNTER DB

## 3.3 Switch State Service Design
### 3.3.1 Orchestration Agent
### 3.3.2 Other Process


## 3.4 SyncD


## 3.5 SAI


## 3.6 User Interface
### 3.6.1 Data Models
A few extensions had to be added to the OpenConfig platform model. Some information has been left out to save space.


```
module: openconfig-platform
  +--rw components
     +--rw component* [name]
        +--rw name                          -> ../config/name
        +--rw config
        |  +--rw name?   string
        +--ro state
        |  +--ro name?                            string
        |  +--ro type?                            union
        |  +--ro id?                              string
        |  +--ro location?                        string
        |  +--ro description?                     string
        |  +--ro mfg-name?                        string
        |  +--ro mfg-date?                        oc-yang:date
        |  +--ro hardware-version?                string
        |  +--ro firmware-version?                string
        |  +--ro software-version?                string
        |  +--ro serial-no?                       string
        |  +--ro part-no?                         string
        |  +--ro removable?                       boolean
        |  +--ro oper-status?                     identityref
        |  +--ro empty?                           boolean
        |  +--ro parent?                          -> ../../../component/config/name
        |  +--ro base-mac-addr?                   oc-yang:mac-address
        |  +--ro temperature
        |  |  +--ro instant?                             decimal64
        |  |  +--ro avg?                                 decimal64
        |  |  +--ro min?                                 decimal64
        |  |  +--ro max?                                 decimal64
        |  |  +--ro interval?                            oc-types:stat-interval
        |  |  +--ro min-time?                            oc-types:timeticks64
        |  |  +--ro max-time?                            oc-types:timeticks64
        |  |  +--ro alarm-status?                        boolean
        |  |  +--ro alarm-threshold?                     uint32
        |  |  +--ro alarm-severity?                      identityref
        |  |  +--ro oc-pf-ext:current?                   decimal64
        |  |  +--ro oc-pf-ext:high-threshold?            decimal64
        |  |  +--ro oc-pf-ext:critical-high-threshold?   decimal64
        |  |  +--ro oc-pf-ext:low-threshold?             decimal64
        |  |  +--ro oc-pf-ext:critical-low-threshold?    decimal64
        |  +--ro memory
        |  |  +--ro available?   uint64
        |  |  +--ro utilized?    uint64
        |  +--ro allocated-power?                 uint32
        |  +--ro used-power?                      uint32
        |  +--ro oc-alarms:equipment-failure?     boolean
        |  +--ro oc-alarms:equipment-mismatch?    boolean
        |  +--ro oc-pf-ext:service-tag?           string
        |  +--ro oc-pf-ext:base-mac-address?      string
        |  +--ro oc-pf-ext:mac-addresses?         int32
        |  +--ro oc-pf-ext:onie-version?          string
        |  +--ro oc-pf-ext:manufacture-country?   string
        |  +--ro oc-pf-ext:vendor-name?           string
        |  +--ro oc-pf-ext:diag-version?          string
        |  +--ro oc-pf-ext:fans?                  uint32
        |  +--ro oc-pf-ext:status-led?            string
        ...
		        +--rw subcomponents
        |  +--rw subcomponent* [name]
        |     +--rw name      -> ../config/name
        |     +--rw config
        |     |  +--rw name?   -> ../../../../../component/config/name
        |     +--ro state
        |        +--ro name?                        -> ../../../../../component/config/name
        |        +--ro oc-pf-ext:sensor-category* [category]
        |           +--ro oc-pf-ext:category    -> ../state/category
        |           +--ro oc-pf-ext:sensors
        |           |  +--ro oc-pf-ext:sensor* [name]
        |           |     +--ro oc-pf-ext:name     -> ../state/name
        |           |     +--ro oc-pf-ext:state
        |           |        +--ro oc-pf-ext:name?    string
        |           |        +--ro oc-pf-ext:state?   string
        |           +--ro oc-pf-ext:state
        |              +--ro oc-pf-ext:category?   string
        ...
		+--rw power-supply
        |  +--rw config
        |  |  +--rw oc-platform-psu:enabled?   boolean
        |  +--ro state
        |     +--ro oc-platform-psu:enabled?          boolean
        |     +--ro oc-platform-psu:capacity?         oc-types:ieeefloat32
        |     +--ro oc-platform-psu:input-current?    oc-types:ieeefloat32
        |     +--ro oc-platform-psu:input-voltage?    oc-types:ieeefloat32
        |     +--ro oc-platform-psu:output-current?   oc-types:ieeefloat32
        |     +--ro oc-platform-psu:output-voltage?   oc-types:ieeefloat32
        |     +--ro oc-platform-psu:output-power?     oc-types:ieeefloat32
        +--rw fan
        |  +--rw config
        |  +--ro state
        |     +--ro oc-pf-ext:direction?         string
        |     +--ro oc-pf-ext:speed-tolerance?   uint32
        |     +--ro oc-pf-ext:target-speed?      uint32
        |     +--ro oc-fan:speed?                uint32
```

### 3.6.2 CLI
There are only show commands for platform information.
#### 3.6.2.1 Configuration Commands
#### 3.6.2.2 Show Commands
```
sonic# show platform
  environment  Show platform Environment
  fanstatus    Show platform fan status
  psustatus    Show platform PSU status
  psusummary   Show platform PSU summary
  syseeprom    Show platform EEPROM information
  temperature  Show platform temperature sensors
  |            Pipe through a command
  <cr>
```
##### Show EERPOM
```
sonic# show platform syseeprom
-----------------------------------------------------------
Attribute            Value/State
-----------------------------------------------------------
description         :x86_64-dell_z9100_c2538-r0
empty               :False
hardware-version    :A02
id                  :Z9100-ON
location            :Slot 1
mfg-date            :2017-08-07
mfg-name            :CES00
name                :System Eeprom
base-mac-address    :34:17:EB:2C:D7:00
diag-version        :3.23.4.1
mac-addresses       :384
manufacture-country :CN
onie-version        :3.23.1.3
service-tag         :847RG02
vendor-name         :DELL
oper-status         :ACTIVE
part-no             :0KY5C4
removable           :False
serial-no           :CN0KY5C4CES007770009
software-version    :'dell_sonic_3.x_share.129-6a2cad56d'

```

##### Show  PSU status
```
sonic# show platform psustatus
-----------------------------------------------------------
PSU                  Status
-----------------------------------------------------------
PSU 1                ACTIVE
PSU 2                INACTIVE

```
##### Show PSU summary
```
sonic# show platform psusummary
PSU 1:
    description          02RPHX
    mfg-name             DELL
    fans                 1
    status-led           green
    oper-status          ACTIVE
    serial-no            CN-02RPHX-17972-74S-018N-A00
    output-current (W)   1.27
    output-power (W)     127.50
    output-voltage (V)   12.19
    fan-speed (RPM)      5808
    fan-direction        fan_direction_exhaust
PSU 2:
    description          02RPHX
    mfg-name             DELL
    fans                 1
    status-led           off
    oper-status          INACTIVE
    serial-no            CN-02RPHX-17972-74I-0139-A00
    fan-speed (RPM)      0
    fan-direction        fan_direction_exhaust
```
##### Show temperature
```
sonic# show platform temperature
TH - Threshold
--------------------------------------------------------------------------------------------------------------
Name                     Temperature    High TH        Low TH         Critical High TH         Critical Low TH
--------------------------------------------------------------------------------------------------------------
ASIC On-board Rear       40.1           85             0              N/A                      N/A
CPU Core 0               40             98             0              N/A                      N/A
CPU Core 1               40             98             0              N/A                      N/A
CPU Core 2               43             98             0              N/A                      N/A
CPU Core 3               42             98             0              N/A                      N/A
CPU On-board             37.3           100            0              N/A                      N/A
System Front Left        24             50             0              N/A                      N/A
System Front Right       23.5           50             0              N/A                      N/A
```
##### Show system sensors
```
sonic# show platform environment

SMF_Z9100_ON-isa-0000
   Adapter: ISA adapter
       BCM Switch On-Board #1 (U38): +30.6 C  (high =  +0.0 C, crit =  +0.0 C)
       BCM Switch On-Board #1 (U44): +40.2 C  (high = +80.0 C, crit = +85.0 C)
       CPU On-board (U2900):         +37.1 C  (high = +95.0 C, crit = +100.0 C)
       CPU VDDR_CPU_1:               +1.34 V
       CPU VDDR_CPU_2:               +1.34 V
       CPU XP0R75V_VTT_A:            +0.66 V
       CPU XP0R75V_VTT_B:            +0.67 V
       CPU XP12R0V:                  +12.08 V
       CPU XP1R07V_CPU:              +1.06 V
       CPU XP1R0V_CPU:               +1.00 V
       CPU XP1R0V_CPU_VCC:           +0.98 V
       CPU XP1R0V_CPU_VNN:           +1.02 V
       CPU XP1R35V_CPU:              +1.34 V
       CPU XP1R5V_CLK:               +1.50 V
       CPU XP1R5V_EARLY:             +1.52 V
       CPU XP1R8V_CPU:               +1.80 V
       CPU XP3R3V_CP:                +3.33 V
       CPU XP3R3V_EARLY:             +3.25 V
       CPU XP3R3V_STD:               +3.30 V
       CPU XP5R0V_CP:                +4.99 V
       Front BCM On-Board (U2):      +22.7 C  (high = +47.0 C, crit = +50.0 C)
	   Front BCM On-Board (U4):      +23.0 C  (high = +47.0 C, crit = +50.0 C)
       PSU1 Input  Current:          +1.40 A
       PSU1 Input  Power:            140.00 W  (max = 750.00 W)
       PSU1 Output Current:          +1.28 A
       PSU1 Output Power:            127.60 W  (max = 750.00 W)
       PSU1 Temp:                    +37.0 C  (high =  +0.0 C, crit =  +0.0 C)
       PSU1 VIN:                     +211.50 V
       PSU1 VOUT:                    +12.19 V
       PSU2 Input  Current:          +0.00 A
       PSU2 Input  Power:            0.00 W  (max = 750.00 W)
       PSU2 Output Current:          +0.00 A
       PSU2 Output Power:            0.00 W  (max = 750.00 W)
       PSU2 Temp:                    +0.0 C  (high =  +0.0 C, crit =  +0.0 C)
       PSU2 VIN:                     +0.00 V
       PSU2 VOUT:                    +0.00 V
       Psu1 Fan:                     5808 RPM
       Psu2 Fan:                     5776 RPM
       Rear (U2900):                 +28.6 C  (high =  +0.0 C, crit =  +0.0 C)
       SW XP1R0V_ROV_SW_MON:         +0.99 V
       SW XP1R0V_SW_MON:             +1.02 V
       SW XP1R25V_MON:               +1.24 V
       SW XP1R2V_MON:                +1.19 V
       SW XP1R8V_FPGA_MON:           +1.78 V
       SW XP1R8V_MON:                +1.78 V
 SW XP3R3V_EARLY_MON:          +3.25 V
       SW XP3R3V_FPGA_MON:           +3.29 V
       SW XP3R3V_MON:                +3.31 V
       SW XP5V_MB_MON:               +4.93 V
       Tray1 Fan1:                   6124 RPM
       Tray1 Fan2:                   6373 RPM
       Tray2 Fan1:                   6172 RPM
       Tray2 Fan2:                   6311 RPM
       Tray3 Fan1:                   6202 RPM
       Tray3 Fan2:                   6404 RPM
       Tray4 Fan1:                   6231 RPM
       Tray4 Fan2:                   6342 RPM
       Tray5 Fan1:                   6202 RPM
       Tray5 Fan2:                   6383 RPM
       XP1R0V:                       +54.00 A
       XP1R0V_ROV:                   +34.00 A


coretemp-isa-0000
   Adapter: ISA adapter
       Core 0:                       +37.0 C  (high = +98.0 C, crit = +98.0 C)
       Core 1:                       +37.0 C  (high = +98.0 C, crit = +98.0 C)
       Core 2:                       +40.0 C  (high = +98.0 C, crit = +98.0 C)
       Core 3:                       +39.0 C  (high = +98.0 C, crit = +98.0 C)
```
#### 3.6.2.3 Debug Commands
#### 3.6.2.4 IS-CLI Compliance

### 3.6.3 REST API Support
```
GET - Get existing platform configuration information from state DB.
```

# 4 Flow Diagrams

# 5 Error Handling


# 6 Serviceability and Debug


# 7 Warm Boot Support


# 8 Scalability


# 9 Unit Test
The unit-test for this feature will include:
#### Configuration via CLI

#### Show platform configuration via CLI
| Test Name | Test Description |
| :------ | :----- |
| show platform EEPROM | Verify EEPROM information in State DB |
| show platform PSU status | Verify PSU status in State DB |
| show platform PSU summary | Verify all PSU information in State DB |
| show platform fan status | Verify fan information in State DB |
| show platform temperature information | Verify temperature information in State DB |
| show platform environment information | Verify system sensor |
#### GET via gNMI

Same test as CLI configuration Test but using gNMI request


#### Get via REST (GET)

Same as CLI show test but with REST GET request, will verify the JSON response is correct.

