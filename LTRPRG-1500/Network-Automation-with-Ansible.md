# **<p align="center">Network Automation with Ansible</p>**
# **<p align="center">LTRPRG-1500</p>**

---
# **<p align="center">Lab Guide</p>**

---
 **<p align="center">Gowtham Tamilselvan | Muthuraja Ayyanar | Yogi Raghunathan </p>**

 **<p align="center">June 10 2019</p>**
---


# Table of Contents


- [1. Ansible Introduction](#1-ansible-introduction)
	- [1.1 Configuration file](#11-configuration-file)
	- [1.2 Inventory file](#12-inventory-file)
	- [1.3 Ansible modules](#13-ansible-modules)
	- [1.4 Ad-hoc commands](#14-ad-hoc-commands)
- [2. Playbook Primer](#2-playbook-primer)
	- [2.1 Raw module](#21-raw-module)
	- [2.2 IOS command module](#22-ios-command-module)
	- [2.3 XR command module](#23-xr-command-module)
	- [2.4 IOS config module](#24-ios-config-module)
	- [2.5 XR config module](#25-xr-config-module)
	- [2.6 Variables](#26-variables)
	- [2.7 Loops](#27-loops)
	- [2.8 Conditionals](#28-conditionals)
	- [2.9 Importing playbooks](#29-importing-playbooks)
- [3. Automating Common Tasks](#3-automating-common-tasks)
	- [3.1 Router config backup](#31-router-config-backup)
	- [3.2 Device health monitoring](#32-device-health-monitoring)
	- [3.3 Method of Procedure (MOP)](#33-method-of-procedure-mop-automation)
	- [3.4 Generate Device Config](#34-generate-device-configuration)
	- [3.5 Bulk Config Generation](#35-bulk-config-generation)
  - [3.6 TextFSM Module](#36-textFSM-module)
  - [3.7 Yogi Custom Module](#37-yogi-custom-module)
- [4. Appendix](#4-appendix)
	- [4.1 Ansible Vault](#41-ansible-vault)
	- [4.2 Optional exercise op23-cmd.yml](#42-optional-exercise-op23-cmdyml)
	- [4.3 Optional exercise op28-conditionals.yml](#43-optional-exercise-op28-conditionalsyml)
	- [4.4 Optional exercise op31-runcfg-bkup.yml (Router Config Backup)](#44-optional-exercise-op31-runcfg-bkupyml-router-config-backup)
	- [4.5 Optional exercise op33-mop.yml (MOP)](#45-optional-exercise-op33-mopyml-mop)
	- [4.6 Ansible installation](#46-ansible-installation)
	- [4.7 Reference](#47-reference)

---

# 1. Ansible introduction

### Objective
- Setup your ansible environment: ansible.cfg & hosts file
- Run Ansible ad-hoc commands

### Lab exercises
- The following topics are covered:
 - Configuration file
 - Inventory file
 - Ansible modules
 - Ad-hoc commands

## 1.1 Configuration file
- Find Ansible config file
  - `$ ansible --version`
  - This output points to `config file = /etc/ansible/ansible.cfg`
- Browse the config file and quickly go over different sections, denoted by []

```
grep -v "#" /etc/ansible/ansible.cfg | grep -v ^$
```
- Above command output shows an empty default config section, you will edit/uncomment the following settings, under [default] section:
  - inventory  = /etc/ansible/hosts
  - gathering = explicit
  - host_key_checking = False
  - timeout = 10
  - deprecation_warnings = False
  - retry_files_enabled = False

- Edit the config file
  - Use your favorite editing method to edit the file
  - Root privileges are required to edit /etc/ansible/ansible.cfg, sudo password is cisco
  - Ubuntu inbuilt editors: vi, vim, or nano
  - [VI reference](./vi-reference.md)

```
sudo vi /etc/ansible/ansible.cfg
```

- The target config lines are already there in the file but are commented. Simply delete # at the beginning of the line.

  - Note: *gathering = implicit* will need to be changed to *explicit*
  - Note: *deprecation_warnings = True* will need to be changed to *False*

- After editing, the config file will look like below.

### Example output

```
cisco@ansible-controller:~$ grep -v "^#" /etc/ansible/ansible.cfg | grep -v ^$

[defaults]
inventory      = /etc/ansible/hosts
gathering = explicit
host_key_checking = False
timeout = 10
deprecation_warnings = False
retry_files_enabled = False
[inventory]
[privilege_escalation]
[paramiko_connection]
[ssh_connection]
[persistent_connection]
[accelerate]
[selinux]
[colors]
[diff]
```
### Reference
> This is for later use. Skip for now. http://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings
> - Changes can be made and used in a configuration file which will be searched for in the following order:
>   - ANSIBLE_CONFIG (environment variable if set)
>   - ./ansible.cfg (in the current directory)
>   - ~/.ansible.cfg (in the home directory)
>   - /etc/ansible/ansible.cfg
> - Ansible will process the above list and use the first file found, all others are ignored.

---

## 1.2 Inventory file
- Edit your default inventory file: /etc/ansible/hosts
- Create two device groups: IOS and XR
- Create one explicit parent group: ALL
- Assign the following variables to the devices: ansible_user=cisco ansible_ssh_pass=cisco
- Find out your IOS and XR router mgmt IP addresses from the pod assignment sheet. Plug them in the file below.
- Edit the hosts file
  - Ubuntu inbuilt editors: vi, vim, or nano
  - Root priviliges are required to edit /etc/ansible/hosts, sudo password is cisco
  - [VI reference](./vi-reference.md)
```
sudo vi /etc/ansible/hosts
```

### Example output
```
cisco@ansible-controller:~$ sudo vi /etc/ansible/hosts

[IOS]
R1 ansible_host=172.16.101.XX ansible_user=cisco ansible_ssh_pass=cisco

[XR]
R2 ansible_host=172.16.101.XX ansible_user=cisco ansible_ssh_pass=cisco

[ALL:children]
IOS
XR
```
- Run the following verification commands:

```
cisco@ansible-controller:~$ ansible --list-hosts IOS
  hosts (1):
    R1
cisco@ansible-controller:~$ ansible --list-hosts XR
  hosts (1):
    R2
cisco@ansible-controller:~$ ansible --list-hosts ALL
  hosts (2):
    R1
    R2
cisco@ansible-controller:~$ ansible --list-hosts all
  hosts (2):
    R2
    R1
```
- By default there are two implicit groups in Ansible: "all" and "ungrouped"
  - "all" contains every host defined in the inventory file
  - "ungrouped" contains all hosts that don't have an explicit group assigned

- Verify the ansible.cfg and hosts files are accurate. (XX should contain numbers for your specific devices)

```
cisco@ansible-controller:~$ grep inventory /etc/ansible/ansible.cfg | grep hosts

inventory      = /etc/ansible/hosts

cisco@ansible-controller:~$ grep -v "#" /etc/ansible/hosts | grep -v ^$

[IOS]
R1	ansible_host=172.16.101.XX ansible_user=cisco ansible_ssh_pass=cisco
[XR]
R2	ansible_host=172.16.101.XX ansible_user=cisco ansible_ssh_pass=cisco
[ALL:children]
IOS
XR

```
### Reference

> - This snippet is for future reference.  http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html
> - Ansible can support multiple inventory files.

---

## 1.3 Ansible modules
- Ansible ships with several modules.
- Try the below commands for a **quick review**

```
$ ansible-doc --help
$ ansible-doc -l | wc -l
$ ansible-doc -l | grep ^ios_
$ ansible-doc -l | grep iosxr
$ ansible-doc -l
$ ansible-doc ios_command
```

### Example output

```
cisco@ansible-controller:~$ ansible-doc --help
Usage: ansible-doc [-l|-F|-s] [options] [-t <plugin type> ] [plugin]

plugin documentation tool

Options:
  -a, --all             **For internal testing only** Show documentation for
                        all plugins.
  -h, --help            show this help message and exit
  -l, --list            List available plugins
  -F, --list_files      Show plugin names and their source files without
                        summaries (implies --list)
  -M MODULE_PATH, --module-path=MODULE_PATH
                        prepend colon-separated path(s) to module library
                        (default=[u'/home/cisco/.ansible/plugins/modules',
                        u'/usr/share/ansible/plugins/modules'])
  -s, --snippet         Show playbook snippet for specified plugin(s)
  -t TYPE, --type=TYPE  Choose which plugin type (defaults to "module")
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
  --version             show program's version number and exit

See man pages for Ansible CLI options or website for tutorials
https://docs.ansible.com

cisco@ansible-controller:~$ ansible-doc -l | wc -l
2080
cisco@ansible-controller:~$ ansible-doc -l | grep ^ios_
ios_banner                                           Manage multiline banners on Cisco IOS devices
ios_command                                          Run commands on remote devices running Cisco IOS
ios_config                                           Manage Cisco IOS configuration sections
ios_facts                                            Collect facts from remote devices running Cisco IOS
ios_interface                                        Manage Interface on Cisco IOS network devices
ios_l2_interface                                     Manage Layer-2 interface on Cisco IOS devices.
ios_l3_interface                                     Manage L3 interfaces on Cisco IOS network devices.
ios_linkagg                                          Manage link aggregation groups on Cisco IOS network devic...
ios_lldp                                             Manage LLDP configuration on Cisco IOS network devices.
ios_logging                                          Manage logging on network devices
ios_ping                                             Tests reachability using ping from Cisco IOS network devi...
ios_static_route                                     Manage static IP routes on Cisco IOS network devices
ios_system                                           Manage the system attributes on Cisco IOS devices
ios_user                                             Manage the aggregate of local users on Cisco IOS device
ios_vlan                                             Manage VLANs on IOS network devices
ios_vrf                                              Manage the collection of VRF definitions on Cisco IOS dev...
cisco@ansible-controller:~$ ansible-doc -l | grep iosxr
iosxr_banner                                         Manage multiline banners on Cisco IOS XR devices
iosxr_command                                        Run commands on remote devices running Cisco IOS XR
iosxr_config                                         Manage Cisco IOS XR configuration sections
iosxr_facts                                          Collect facts from remote devices running IOS XR
iosxr_interface                                      Manage Interface on Cisco IOS XR network devices
iosxr_logging                                        Configuration management of system logging services on ne...
iosxr_netconf                                        Configures NetConf sub-system service on Cisco IOS-XR dev...
iosxr_system                                         Manage the system attributes on Cisco IOS XR devices
iosxr_user                                           Manage the aggregate of local users on Cisco IOS XR devic...
cisco@ansible-controller:~$ ansible-doc -l | grep iosxr | wc -l
9
cisco@ansible-controller:~$
```
### Reference

> For future research, check the ansible-docs page to find details on available modules.
> - http://docs.ansible.com/ansible/latest/modules/modules_by_category.html

---

## 1.4 Ad-hoc commands
- Let's use a few modules in this section: `raw`, `ios_command`, and `iosxr_command`
- Syntax: `ansible <devices> -m <module> -a <command>`
	- devices must exist in the inventory file
- Execute the below ad-hoc commands (from your home directory, /home/cisco)

```
$ ansible IOS -m raw -a "show ip int brief"
$ ansible ALL -m raw -a "show clock"
$ ansible IOS --connection local -m ios_command -a "commands='show ip route summ'"
$ ansible XR --connection local -m iosxr_command -a "commands='show route summ'"
```
- Raw is a barebones module that simply executes a command via an SSH connection and returns the resulting text.
- The ios_command module provides additional functionality over raw, such as specifying multiple commands to collect and returning metadata about the task execution (ex: failed and changed statuses).
- Note: ios_command and iosxr_command require the "--connection local" flag.  Networking modules do not use the default connection types (SSH) because most network hosts run limited execution environments that do not provide native Python.  Ansible modules using SSH connection type rely on Python scripts to be transferred and executed on the target node, whereas network modules are executed on the Ansible host (locally) and use other connection providers to interact with the target system.

### Optional exercises

> - Do this section if you are ahead of schedule, else skip and come back later.
> - This lab will be available to you for a few days after today's session.
> - It is possible to use ad-hoc commands with more arguments, as below:
> ```
> $ ansible IOS -c local -m ios_command -a "authorize=true commands='show run int gig1'"
> $ ansible IOS -c local -m ios_command -a "username=cisco password=cisco auth_pass=cisco authorize=true commands='show run int gig1'"
> $ ansible XR -c local -m iosxr_facts -a "username=cisco password=cisco gather_subset=hardware"
> ```

### Conclusion

- You are now able to use ad-hoc ansible commands to print outputs from the two routers.
- Review the section and discuss if you have any questions.


### Example output

```
cisco@ansible-controller:~$ ansible IOS -m raw -a "show ip int bri"
R1 | CHANGED | rc=0 >>

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       172.16.101.191  YES TFTP   up                    up      
GigabitEthernet2       10.0.0.5        YES TFTP   up                    up      
Loopback0              192.168.0.1     YES TFTP   up                    up      Shared connection to 172.16.101.191 closed.


cisco@ansible-controller:~$ ansible ALL -m raw -a "show clock"
R1 | CHANGED | rc=0 >>

*01:04:55.419 UTC Tue Jan 8 2019Shared connection to 172.16.101.191 closed.


R2 | CHANGED | rc=0 >>


Tue Jan  8 01:05:02.562 UTC
01:05:02.612 UTC Tue Jan 8 2019

:

cisco@ansible-controller:~$ ansible IOS --connection local -m ios_command -a "commands='show ip route summ'"
R1 | SUCCESS => {
    "changed": false,
    "stdout": [
        "IP routing table name is default (0x0)\nIP routing table maximum-paths is 32\nRoute Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)\napplication     0           0           0           0           0\nconnected       0           3           0           288         912\nstatic          0           0           0           0           0\ninternal        2                                               968\nTotal           2           3           0           288         1880"
    ],
    "stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "internal        2                                               968",
            "Total           2           3           0           288         1880"
        ]
    ]
}
cisco@ansible-controller:~$ ansible XR --connection local -m iosxr_command -a "commands='show route summ'"
R2 | SUCCESS => {
    "changed": false,
    "stdout": [
        "Route Source                     Routes     Backup     Deleted     Memory(bytes)\nconnected                        1          1          0           320          \nlocal                            2          0          0           320          \ndagr                             0          0          0           0            \nTotal                            3          1          0           640"
    ],
    "stdout_lines": [
        [
            "Route Source                     Routes     Backup     Deleted     Memory(bytes)",
            "connected                        1          1          0           320          ",
            "local                            2          0          0           320          ",
            "dagr                             0          0          0           0            ",
            "Total                            3          1          0           640"
        ]
    ]
}
```

---

# 2. Playbook primer
- Ansible playbook exercises require creating YAML style playbook files.
- Use Ubuntu's "vi" or "vim" or "nano" editor, which is included in your Ansible controller to create YAML playbook files.

- The following topics are covered in this section:
  - Raw Module
  - IOS Command Module
  - XR Command Module
  - IOS Config Module
  - XR Config Module
  - Variables
  - Loops
  - Conditionals
  - Importing Playbooks

## 2.1 Raw module
### Lab exercise

- Use the raw module in a playbook.
- Create a playbook file (p1-raw.yml) with the below content, in your home directory.

```
cisco@ansible-controller:~$ pwd
/home/cisco

cisco@ansible-controller:~$ vi p1-raw.yml
---
- name: get interface info from all hosts
  hosts: ALL

  tasks:
    - name: execute show ip interface brief
      raw:
        show ip interface brief
```
- Predict the outcome of this playbook.
- Execute the play book using the commands below.
  - Note: It is good practice to use the --syntax-check option first to ensure the playbook does not contain any syntax errors before execution.
  - Note: The Syntax check output will be blank if there are no errors.  Output is expected only if errors are present.

```
$ ansible-playbook p1-raw.yml --syntax-check

$ ansible-playbook p1-raw.yml

$ ansible-playbook p1-raw.yml -v
```

- As you may have noticed, command output is not displayed unless you run it with -v verbose mode.
- Create another playbook p1a-raw.yml, to always print the output from the routers.

```
cisco@ansible-controller:~$ vi p1a-raw.yml
---
- name: get interface info from all hosts
  hosts: ALL

  tasks:
    - name: execute show ip interface brief
      raw:
        show ip interface brief

      register: P2_RAW_OUTPUT

    - name: print data saved in the variable
      debug:
        var: P2_RAW_OUTPUT.stdout_lines
```
- Predict the outcome of this playbook.
- Execute the playbook
  - Note: --step will cause Ansible to prompt you before executing each task. Answer with "Y" to execute the task, with "N" to skip the task, and "c" to continue the rest of the playbook without further prompts

```
$ ansible-playbook p1a-raw.yml --syntax-check

$ ansible-playbook p1a-raw.yml

$ ansible-playbook p1a-raw.yml --step

$ ansible-playbook p1a-raw.yml -v
```

### Conclusion

- In this section you used the raw module to collect and display command output from devices that are in the group, named ALL.
- Review the section and discuss if you have any questions.


### Example output

```
cisco@ansible-controller:~$ ansible-playbook p1-raw.yml --syntax-check

playbook: p1-raw.yml
cisco@ansible-controller:~$ ansible-playbook p1-raw.yml

PLAY [get interface info from all hosts] **************************************************************************************************************************

TASK [execute show ip interface brief] ****************************************************************************************************************************
changed: [R1]
changed: [R2]

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=1    changed=1    unreachable=0    failed=0   
R2                         : ok=1    changed=1    unreachable=0    failed=0   


cisco@ansible-controller:~$ ansible-playbook p1a-raw.yml --syntax-check

playbook: p1a-raw.yml
cisco@ansible-controller:~$ ansible-playbook p1a-raw.yml

PLAY [get interface info from all hosts] **************************************************************************************************************************

TASK [execute show ip interface brief] ****************************************************************************************************************************
changed: [R1]
changed: [R2]

TASK [print data saved in the variable] ***************************************************************************************************************************
ok: [R2] => {
    "P2_RAW_OUTPUT.stdout_lines": [
        "",
        "",
        "Tue Jan  8 01:20:08.420 UTC",
        "",
        "Interface                      IP-Address      Status          Protocol Vrf-Name",
        "Loopback0                      192.168.0.2     Up              Up       default ",
        "MgmtEth0/0/CPU0/0              172.16.101.192  Up              Up       Mgmt-intf",
        "GigabitEthernet0/0/0/0         10.0.0.6        Up              Up       default "
    ]
}
ok: [R1] => {
    "P2_RAW_OUTPUT.stdout_lines": [
        "",
        "Interface              IP-Address      OK? Method Status                Protocol",
        "GigabitEthernet1       172.16.101.191  YES TFTP   up                    up      ",
        "GigabitEthernet2       10.0.0.5        YES TFTP   up                    up      ",
        "Loopback0              192.168.0.1     YES TFTP   up                    up      "
    ]
}

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=2    changed=1    unreachable=0    failed=0   
R2                         : ok=2    changed=1    unreachable=0    failed=0   

```

---

## 2.2 IOS command module
### Lab exercise
- Use the ios_command to execute some exec level commands on IOS devices.
- Create a playbook, p2-ioscmd.yml, with the below content, in your home directory.

```
cisco@ansible-controller:~$ vi p2-ioscmd.yml
---
- name: collect ip route summary from all IOS devices
  hosts: IOS
  connection: local

  tasks:
    - name: execute route summary command
      ios_command:
        commands:
          show ip route summary

      register: P2_OUTPUT

    - name: print output
      debug:
        var: P2_OUTPUT.stdout_lines
```
- Predict the outcome of this playbook.
- Execute the playbook
- After reviewing the playbook output, try run the playbook in verbose mode: with -v, -vv, or -vvv

```
$ ansible-playbook p2-ioscmd.yml --syntax-check

$ ansible-playbook p2-ioscmd.yml
```

### Conclusion
- In this section you used the ios_command module to collect and display command output from an IOS device.
- Review the section and discuss if you have any questions.

### Reference

> Reference:
> - this is for future reference only (don't spend time on this now).
> - Pay attention to sections: parameters and return values.
> - http://docs.ansible.com/ansible/latest/modules/ios_command_module.html
> - http://docs.ansible.com/ansible/latest/modules/modules_by_category.html


### Example output

```
cisco@ansible-controller:~$ ansible-playbook p2-ioscmd.yml

PLAY [collect ip route summary from all IOS devices] **************************************************************************************************************

TASK [execute route summary command] ******************************************************************************************************************************
ok: [R1]

TASK [print output] ***********************************************************************************************************************************************
ok: [R1] => {
    "P2_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "internal        2                                               968",
            "Total           2           3           0           288         1880"
        ]
    ]
}

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0
```

---

## 2.3 XR command module
### Lab exercise
- Use the iosxr_command to execute some exec level commands on XR devices.
- Create a playbook, p3-xrcmd.yml, with the below content, in your home directory.

```
cisco@ansible-controller:~$ vi p3-xrcmd.yml
---
- name: collect ip route summary from all XR devices
  hosts: XR
  connection: local

  tasks:
    - name: execute route summary command
      iosxr_command:
        commands:
          show route summary

      register: P3_OUTPUT

    - name: print output
      debug:
        var: P3_OUTPUT.stdout_lines
```
- Predict the outcome of this playbook.
- Execute the playbook

```
$ ansible-playbook p3-xrcmd.yml --syntax-check
$ ansible-playbook p3-xrcmd.yml
```

### Conclusion
- In this section you used the xr_command module to collect and display command output from an XR router.
- Review the section and discuss if you have any questions.

### Optional exercise
> - Do this section if you are ahead of schedule, else skip and come back later.
> - This lab will be available for you to work for a few days after Cisco Live. You have the option of doing this later as well.
> - Write a playbook, op23-cmd.yml, to meet the below requirements:
>   - Collect output of route summary from both IOS and XR routers
>   - Use the modules, ios_command and iosxr_command
>   - Write 2 plays within one playbook.
> - Playbook solution is in the appendix section 4.2 (op23-cmd.yml).

### Example output
```
cisco@ansible-controller:~$ ansible-playbook p3-xrcmd.yml --syntax-check

playbook: p3-xrcmd.yml
cisco@ansible-controller:~$ ansible-playbook p3-xrcmd.yml

PLAY [collect ip route summary from all XR devices] ***************************************************************************************************************

TASK [execute route summary command] ******************************************************************************************************************************
ok: [R2]

TASK [print output] ***********************************************************************************************************************************************
ok: [R2] => {
    "P3_OUTPUT.stdout_lines": [
        [
            "Route Source                     Routes     Backup     Deleted     Memory(bytes)",
            "connected                        1          1          0           320          ",
            "local                            2          0          0           320          ",
            "dagr                             0          0          0           0            ",
            "Total                            3          1          0           640"
        ]
    ]
}

PLAY RECAP ********************************************************************************************************************************************************
R2                         : ok=2    changed=0    unreachable=0    failed=0  
```

---

## 2.4 IOS config module
### Lab exercise
- Use ios_config module to configure a loopback interface on an IOS router.
- Create a playbook, p4-iosconfig.yml, with the below content, on your home directory.

```
cisco@ansible-controller:~$ vi p4-iosconfig.yml
---
- name: configure loopback1 interface on IOS devices
  hosts: IOS
  connection: local

  tasks:
    - name: configure loopback101 interface
      ios_config:
        parents: interface loopback101
        lines:
          - description test config by p4
          - ip address 1.1.1.101 255.255.255.255
          - shutdown
```

- Verify loopback101 interface does not already exist
- Execute the playbook
- Check if loopback101 interface is created by p4-iosconfig.yml playbook

```
$ ansible IOS -c local -m ios_command -a "commands='show run int loop101'"

$ ansible-playbook p4-iosconfig.yml --syntax-check

$ ansible-playbook p4-iosconfig.yml

$ ansible IOS -c local -m ios_command -a "commands='show run int loop101'"
```

### Conclusion
-  In this section you used the ios_config module to configure a loopback interface on an IOS router.
- Review the section and discuss if you have any questions

### Example output
```
cisco@ansible-controller:~$ ansible IOS -c local -m ios_command -a "commands='show run int loop101'"
R1 | FAILED! => {
    "changed": false,
    "msg": "show run int loop101\r\n                          ^\r\n% Invalid input detected at '^' marker.\r\n\r\nR1-CSR1K#"
}

cisco@ansible-controller:~$ ansible-playbook p4-iosconfig.yml --syntax-check

playbook: p4-iosconfig.yml

cisco@ansible-controller:~$ ansible-playbook p4-iosconfig.yml

PLAY [configure loopback1 interface on IOS devices] ***************************************************************************************************************

TASK [configure loopback101 interface] ****************************************************************************************************************************
changed: [R1]

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=1    changed=1    unreachable=0    failed=0   

cisco@ansible-controller:~$ ansible IOS -c local -m ios_command -a "commands='show run int loop101'"
R1 | SUCCESS => {
    "changed": false,
    "stdout": [
        "Building configuration...\n\nCurrent configuration : 108 bytes\n!\ninterface Loopback101\n description test config by p4\n ip address 1.1.1.101 255.255.255.255\n shutdown\nend"
    ],
    "stdout_lines": [
        [
            "Building configuration...",
            "",
            "Current configuration : 108 bytes",
            "!",
            "interface Loopback101",
            " description test config by p4",
            " ip address 1.1.1.101 255.255.255.255",
            " shutdown",
            "end"
        ]
    ]
}
```
---

## 2.5 XR config module
### Lab exercise
- Use xr_config module to configure an access-list on a XR router.
- Create a playbook, p5-xrconfig.yml, with the below content, on your home directory.

```
cisco@ansible-controller:~$ vi p5-xrconfig.yml
---
- name: configure ACL test7 on all XR devices
  hosts: XR
  connection: local

  tasks:
    - name: configure acl test7
      iosxr_config:
        parents: ipv4 access-list test7
        lines:
          - 10 permit ipv4 host 1.1.1.1 any
          - 20 permit ipv4 host 2.2.2.2 any
          - 30 permit ipv4 host 3.3.3.3 any
```

- Predict the outcome of executing the above playbook
- Check the ACL before configuring
- Run the playbook
- Check for the post-playbook config

```
$ ansible XR -c local -m iosxr_command -a "commands='show run ipv4 access-list'"

$ ansible-playbook p5-xrconfig.yml --syntax-check

$ ansible-playbook p5-xrconfig.yml

$ ansible XR -c local -m iosxr_command -a "commands='show run ipv4 access-list'"
```

### Conclusion
- In this section you used the xr_config module to configure an access-list on an XR router.
- Review the section and discuss if you have any questions

### Example output
```
cisco@ansible-controller:~$ ansible XR -c local -m iosxr_command -a "commands='show run ipv4 access-list'"
R2 | SUCCESS => {
    "changed": false,
    "stdout": [
        "% No such configuration item(s)"
    ],
    "stdout_lines": [
        [
            "% No such configuration item(s)"
        ]
    ]
}

cisco@ansible-controller:~$ ansible-playbook p5-xrconfig.yml --syntax-check

playbook: p5-xrconfig.yml

cisco@ansible-controller:~$ ansible-playbook p5-xrconfig.yml

PLAY [configure ACL test7 on all XR devices] **********************************************************************************************************************

TASK [configure acl test7] ****************************************************************************************************************************************
changed: [R2]

PLAY RECAP ********************************************************************************************************************************************************
R2                         : ok=1    changed=1    unreachable=0    failed=0   

cisco@ansible-controller:~$ ansible XR -c local -m iosxr_command -a "commands='show run ipv4 access-list'"
R2 | SUCCESS => {
    "changed": false,
    "stdout": [
        "ipv4 access-list test7\n 10 permit ipv4 host 1.1.1.1 any\n 20 permit ipv4 host 2.2.2.2 any\n 30 permit ipv4 host 3.3.3.3 any\n!"
    ],
    "stdout_lines": [
        [
            "ipv4 access-list test7",
            " 10 permit ipv4 host 1.1.1.1 any",
            " 20 permit ipv4 host 2.2.2.2 any",
            " 30 permit ipv4 host 3.3.3.3 any",
            "!"
        ]
    ]
}
```

---

## 2.6 Variables
- Ansible uses variables to enable more flexibility in playbooks.
- Variables are used to store information. This information can be used in a playbook by calling the specific variable.

### Lab exercise
- Create custom variables inside a playbook to store an interface name and call the variable to complete the show command execution.
- Create a playbook, p6-vars.yml, with the below content, on your home directory.

```
cisco@ansible-controller:~$ vi p6-vars.yml
---
- name: get config of gig1 and gig2
  hosts: IOS
  connection: local
  vars:
    INTF1: GigabitEthernet1
    INTF2: GigabitEthernet2

  tasks:
    - name: Show run interface Gig
      ios_command:
        commands:
          - sho run int {{INTF1}}
          - sho run int {{INTF2}}

      register: INT_OUT

    - name: print stuff, style-1
      debug:
        var: INT_OUT

    - name: print stuff, style-2
      debug:
        var: INT_OUT.stdout_lines[0]

    - name: print stuff, style-2
      debug:
        var: INT_OUT.stdout_lines[1]
```
- Predict the outcome of executing the above playbook
- Run the playbook

```
$ ansible-playbook p6-vars.yml --syntax-check

$ ansible-playbook p6-vars.yml
```


### Conclusion
- In this section you created variables inside a playbook and recalled the variables to complete a task execution.
- When multiple commands are given to the ios_command module, the results of each command output are stored in individual \<variable\>.stdout_lines array items.  They can be recalled individually by referencing their order in the array.  Ex:  \<variable\>.stdout_lines[n]
- Review the section and discuss if you have any questions.

### Reference
> - This is a simple exercise on variables; recommend to read below pages for more advanced use cases.
> - http://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html

### Example output
```
cisco@ansible-controller:~$ ansible-playbook p6-vars.yml --syntax-check

playbook: p6-vars.yml
cisco@ansible-controller:~$ ansible-playbook p6-vars.yml

PLAY [get config of gig1 and gig2] ********************************************************************************************************************************

TASK [Show run interface Gig] *************************************************************************************************************************************
ok: [R1]

TASK [print stuff, style-1] ***************************************************************************************************************************************
ok: [R1] => {
    "INT_OUT": {
        "changed": false,
        "failed": false,
        "stdout": [
            "Building configuration...\n\nCurrent configuration : 189 bytes\n!\ninterface GigabitEthernet1\n description OOB Management\n vrf forwarding Mgmt-intf\n ip address 172.16.101.191 255.255.255.0\n negotiation auto\n cdp enable\n no mop enabled\n no mop sysid\nend",
            "Building configuration...\n\nCurrent configuration : 170 bytes\n!\ninterface GigabitEthernet2\n description to R2-XRv\n ip address 10.0.0.5 255.255.255.252\n ip ospf cost 1\n negotiation auto\n cdp enable\n no mop enabled\n no mop sysid\nend"
        ],
        "stdout_lines": [
            [
                "Building configuration...",
                "",
                "Current configuration : 189 bytes",
                "!",
                "interface GigabitEthernet1",
                " description OOB Management",
                " vrf forwarding Mgmt-intf",
                " ip address 172.16.101.191 255.255.255.0",
                " negotiation auto",
                " cdp enable",
                " no mop enabled",
                " no mop sysid",
                "end"
            ],
            [
                "Building configuration...",
                "",
                "Current configuration : 170 bytes",
                "!",
                "interface GigabitEthernet2",
                " description to R2-XRv",
                " ip address 10.0.0.5 255.255.255.252",
                " ip ospf cost 1",
                " negotiation auto",
                " cdp enable",
                " no mop enabled",
                " no mop sysid",
                "end"
            ]
        ]
    }
}

TASK [print stuff, style-2] ***************************************************************************************************************************************
ok: [R1] => {
    "INT_OUT.stdout_lines[0]": [
        "Building configuration...",
        "",
        "Current configuration : 189 bytes",
        "!",
        "interface GigabitEthernet1",
        " description OOB Management",
        " vrf forwarding Mgmt-intf",
        " ip address 172.16.101.191 255.255.255.0",
        " negotiation auto",
        " cdp enable",
        " no mop enabled",
        " no mop sysid",
        "end"
    ]
}

TASK [print stuff, style-2] ***************************************************************************************************************************************
ok: [R1] => {
    "INT_OUT.stdout_lines[1]": [
        "Building configuration...",
        "",
        "Current configuration : 170 bytes",
        "!",
        "interface GigabitEthernet2",
        " description to R2-XRv",
        " ip address 10.0.0.5 255.255.255.252",
        " ip ospf cost 1",
        " negotiation auto",
        " cdp enable",
        " no mop enabled",
        " no mop sysid",
        "end"
    ]
}

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=4    changed=0    unreachable=0    failed=0    

```

---

## 2.7 Loops
- Loops are used to perform a task repeatedly with a set of different items.

### Lab exercise
- Utilize Ansible loops to simplify execution of multiple show commands.
- Create a playbook, p7-loops.yml, with the below content, on your home directory.

```
cisco@ansible-controller:~$ vi p7-loops.yml
---
- name: get config of gig1 and gig2 from IOS devices
  hosts: IOS
  connection: local

  tasks:
    - name: get config of gig1 and gig2 and time
      ios_command:
        commands:
          - "{{item}}"

      with_items:
           - show run int gig1
           - show run int gig2
           - show clock

      register: P7_OUT

    - name: print stuff
      debug:
        var: P7_OUT
```
- Predict the outcome of executing the above playbook
- Run the playbook

```
$ ansible-playbook p7-loops.yml --syntax-check

$ ansible-playbook p7-loops.yml -v
```
- Note the Loop command outputs will print a list of variables set to null, these are all other variables/parameters available within this module.

### Conclusion
- In this section you created a loop to iterate the execution of multiple show commands.
- Review the section and discuss if you have any questions


### Reference

> Reference: http://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html


### Example output
```
cisco@ansible-controller:~$ ansible-playbook p7-loops.yml --syntax-check

playbook: p7-loops.yml

cisco@ansible-controller:~$ ansible-playbook p7-loops.yml {#reduced output}

PLAY [get config of gig1 and gig2 from IOS devices] **************************************************

TASK [get config of gig1 and gig2 and time] **********************************************************
ok: [R1] => (item=show run int gig1)
ok: [R1] => (item=show run int gig2)
ok: [R1] => (item=show clock)

TASK [print stuff] ***********************************************************************************
ok: [R1] => {
    "P7_OUT": {
        "changed": false,
        "msg": "All items completed",
        "results": [
            {
            :
                },
                "item": "show run int gig1",
                "stdout": [
                    "Building configuration...\n\nCurrent configuration : 188 bytes\n!\ninterface GigabitEthernet1\n description OOB Management\n vrf forwarding Mgmt-intf\n ip address 172.16.101.191 255.255.255.0\n negotiation auto\n cdp enable\n no mop enabled\n no mop sysid\nend"
                ],
                "stdout_lines": [
                    [
                        "Building configuration...",
                        "",
                        "Current configuration : 188 bytes",
                        "!",
                        "interface GigabitEthernet1",
                        " description OOB Management",
                        " vrf forwarding Mgmt-intf",
                        " ip address 172.16.101.191 255.255.255.0",
                        " negotiation auto",
                        " cdp enable",
                        " no mop enabled",
                        " no mop sysid",
                        "end"
                    ]
                ]
            },
            {
              :
                  }
                },
                "item": "show run int gig2",
                "stdout": [
                    "Building configuration...\n\nCurrent configuration : 177 bytes\n!\ninterface GigabitEthernet2\n description Connected to XRV\n ip address 10.0.0.5 255.255.255.252\n ip ospf cost 1\n negotiation auto\n cdp enable\n no mop enabled\n no mop sysid\nend"
                ],
                "stdout_lines": [
                    [
                        "Building configuration...",
                        "",
                        "Current configuration : 177 bytes",
                        "!",
                        "interface GigabitEthernet2",
                        " description Connected to XRV",
                        " ip address 10.0.0.5 255.255.255.252",
                        " ip ospf cost 1",
                        " negotiation auto",
                        " cdp enable",
                        " no mop enabled",
                        " no mop sysid",
                        "end"
                    ]
                ]
            },
            {
            :
                }
                },
                "item": "show clock",
                "stdout": [
                    "*02:21:31.102 UTC Tue Jan 8 2019"
                ],
                "stdout_lines": [
                    [
                        "*02:21:31.102 UTC Tue Jan 8 2019"
                    ]
                ]
            }
        ]
    }
}

PLAY RECAP *******************************************************************************************
R1              : ok=2    changed=0    unreachable=0    failed=0

cisco@ansible-controller:~$
```

---

## 2.8 Conditionals

- Conditionals are used, to decide whether to run a task or not.
- In this section, you will be working on "when" condition.

### Lab exercise
- Collect route summary data by using appropriate command based on the router’s OS
- Create a playbook, p8-conditionals.yml, with the below content, on your home directory.

```
cisco@ansible-controller:~$ vi p8-conditionals.yml
---
- name: get route summary from IOS and XR routers
  hosts: ALL

  tasks:
    - name: collect version info
      raw: show version

      register: SHVER

    - name: run "show ip route summ" on IOS routers
      when: '"IOS XE" in SHVER.stdout'
      raw: show ip route summary

      register: IOSRT

    - debug: var=IOSRT.stdout_lines
      when: IOSRT.stdout_lines is defined

    - name: run "show route summ" on XR routers
      when: '"IOS XR" in SHVER.stdout'
      raw: show route summary

      register: XRRT

    - debug: var=XRRT.stdout_lines
      when: XRRT.stdout_lines is defined
```

- Predict the outcome of executing the above playbook
- Run the playbook

```
$ ansible-playbook p8-conditionals.yml --syntax-check

$ ansible-playbook p8-conditionals.yml -v
```

### Conclusion
- In this section you created a conditional statement to use a show command based on the Router OS.
- Review the section and discuss if you have any questions.

### Optional exercise
> - Create a playbook, which will:
>   - Detect router OS
>   - If a router has IOS, print message, "\<hostname\> is a IOS router" and if a router has XR, print, "\<hostname\> is a XR router"
>   - Playbook needs to find the router names dynamically from the inventory file.
> - Solution playbook is included in the appendix section 4.3 (op28-conditionals.yml).


### Reference
> Reference: http://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html#the-when-statement


### Example output
```
cisco@ansible-controller:~$ ansible-playbook p8-conditionals.yml --syntax-check

playbook: p8-conditionals.yml

cisco@ansible-controller:~$ ansible-playbook p8-conditionals.yml

PLAY [get route summary from IOS and XR routers] ******************************************************************************************************************

TASK [collect version info] ***************************************************************************************************************************************
changed: [R1]
changed: [R2]

TASK [run "show ip route summ" on IOS routers] ********************************************************************************************************************
skipping: [R2]
changed: [R1]

TASK [debug] ******************************************************************************************************************************************************
ok: [R2] => {
    "IOSRT.stdout_lines": "VARIABLE IS NOT DEFINED!"
}
ok: [R1] => {
    "IOSRT.stdout_lines": [
        "",
        "IP routing table name is default (0x0)",
        "IP routing table maximum-paths is 32",
        "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
        "application     0           0           0           0           0",
        "connected       0           3           0           288         912",
        "static          0           0           0           0           0",
        "internal        2                                               968",
        "Total           2           3           0           288         1880"
    ]
}

TASK [run "show route summ" on XR routers] ************************************************************************************************************************
skipping: [R1]
changed: [R2]

TASK [debug] ******************************************************************************************************************************************************
ok: [R2] => {
    "XRRT.stdout_lines": [
        "",
        "",
        "Tue Jan  8 02:32:48.732 UTC",
        "Route Source                     Routes     Backup     Deleted     Memory(bytes)",
        "connected                        1          1          0           320          ",
        "local                            2          0          0           320          ",
        "dagr                             0          0          0           0            ",
        "Total                            3          1          0           640          ",
        ""
    ]
}
ok: [R1] => {
    "XRRT.stdout_lines": "VARIABLE IS NOT DEFINED!"
}

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=4    changed=2    unreachable=0    failed=0   
R2                         : ok=4    changed=2    unreachable=0    failed=0   

```

---

## 2.9 Importing playbooks
- Ansible allows for a playbook to import and execute another playbook using the import_playbook module.
- You will be calling (or importing) playbooks using the import_playbook module.

### Lab exercise
- Create a playbook, p9-import.yml, with the below contents

```
cisco@ansible-controller:~$ vi p9-import.yml
---
- name: route summary from IOS routers
  import_playbook: p2-ioscmd.yml

- name: route summary from XR routers
  import_playbook: p3-xrcmd.yml
```

- Read the playbooks p2-ioscmd.yml and p3-xrcmd.yml
- Predict the outcome of executing the playbook, p9-import.yml
- Run the playbook

```
$ cat p2-ioscmd.yml

$ cat p3-xrcmd.yml

$ ansible-playbook p9-import.yml --syntax-check

$ ansible-playbook p9-import.yml -vv
```

### Conclusion
- Note that import_playbook is to be used at **play-level** and not at task-level
- In this section you created a playbook to import and execute two other playbooks, rather than recreating the same playbook content again.
- Review the section and discuss if you have any questions.

### Reference
> Details: https://docs.ansible.com/ansible/2.4/import_playbook_module.html
>

### Example output

```
cisco@ansible-controller:~$ ansible-playbook p9-import.yml -vv
ansible-playbook 2.7.5
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/cisco/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 2.7.15rc1 (default, Nov 12 2018, 14:31:15) [GCC 7.3.0]
Using /etc/ansible/ansible.cfg as config file
/etc/ansible/hosts did not meet host_list requirements, check plugin documentation if this is unexpected
/etc/ansible/hosts did not meet script requirements, check plugin documentation if this is unexpected

PLAYBOOK: p9-import.yml *******************************************************************************************************************************************
2 plays in p9-import.yml

PLAY [collect ip route summary from all IOS devices] **************************************************************************************************************
META: ran handlers

TASK [execute route summary command] ******************************************************************************************************************************
task path: /home/cisco/p2-ioscmd.yml:7
ok: [R1] => {"changed": false, "stdout": ["IP routing table name is default (0x0)\nIP routing table maximum-paths is 32\nRoute Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)\napplication     0           0           0           0           0\nconnected       0           3           0           288         912\nstatic          0           0           0           0           0\ninternal        2                                               968\nTotal           2           3           0           288         1880"], "stdout_lines": [["IP routing table name is default (0x0)", "IP routing table maximum-paths is 32", "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)", "application     0           0           0           0           0", "connected       0           3           0           288         912", "static          0           0           0           0           0", "internal        2                                               968", "Total           2           3           0           288         1880"]]}

TASK [print output] ***********************************************************************************************************************************************
task path: /home/cisco/p2-ioscmd.yml:14
ok: [R1] => {
    "P2_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "internal        2                                               968",
            "Total           2           3           0           288         1880"
        ]
    ]
}
META: ran handlers
META: ran handlers

PLAY [collect ip route summary from all XR devices] ***************************************************************************************************************
META: ran handlers

TASK [execute route summary command] ******************************************************************************************************************************
task path: /home/cisco/p3-xrcmd.yml:7
ok: [R2] => {"changed": false, "stdout": ["Route Source                     Routes     Backup     Deleted     Memory(bytes)\nconnected                        1          1          0           320          \nlocal                            2          0          0           320          \ndagr                             0          0          0           0            \nTotal                            3          1          0           640"], "stdout_lines": [["Route Source                     Routes     Backup     Deleted     Memory(bytes)", "connected                        1          1          0           320          ", "local                            2          0          0           320          ", "dagr                             0          0          0           0            ", "Total                            3          1          0           640"]]}

TASK [print output] ***********************************************************************************************************************************************
task path: /home/cisco/p3-xrcmd.yml:14
ok: [R2] => {
    "P3_OUTPUT.stdout_lines": [
        [
            "Route Source                     Routes     Backup     Deleted     Memory(bytes)",
            "connected                        1          1          0           320          ",
            "local                            2          0          0           320          ",
            "dagr                             0          0          0           0            ",
            "Total                            3          1          0           640"
        ]
    ]
}
META: ran handlers
META: ran handlers

PLAY RECAP ********************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0   
R2                         : ok=2    changed=0    unreachable=0    failed=0   

```

---

# 3. Automating common tasks
- The following exercises will use Ansible to automate certain network operations tasks:
  - Router Configuration Backup
  - Device Health Monitoring
  - Method of Procedure Automation
  - Generate iBGP Config using Roles
  - Generating bulk configuration
  - TextFSM

---
## 3.1 Router config backup
### Objective
- Create a playbook to capture and backup a router’s running config.

### Approach
- Create a playbook to collect the running config and save it to file on the ansible controller.
- Setup a cron job to execute the playbook once a day and automatically backup the router's running-config.

### Lab exercise

#### Step-1: Create an ansible playbook with a single play which will collect "show run" output from all the nodes.

```
cisco@Ansible-Controller:~/project1$ vi p31-runcfg-bkup.yml
---
- name: Get Router Config from All Routers
  hosts: all
  gather_facts: no

  tasks:
    - name: Collect Show run from all routers
      raw: "show run"

      register: RUNCFG
```

Note when using the raw module do not set the connection to local. The raw module simply takes the input argument and executes the command on the remote host.

#### Step-2: Edit the previous play to add a new task which will perform a time lookup. Then add another task to save the output to a file using the copy module.

Use the set_fact option to set a variable "time" equal to the current time value on the server.

```
  tasks:
    - name: Collect Show run from all routers
      raw: "show run"

      register: RUNCFG

    - set_fact: time="{{lookup('pipe','date \"+%Y-%m-%d-%H-%M\"')}}"

    - name: save output to a file
      connection: local
      copy:
        content="\n ===show run=== \n {{ RUNCFG.stdout }}"
        dest="./{{ inventory_hostname }}_run_cfg_{{ time }}.txt"

```

Note when saving the output to a file set the connection back to local, since the output files need to be saved on the ansible controller.

- Run the playbook
- Look for config files in `/home/cisco` directory

```
$ ansible-playbook p31-runcfg-bkup.yml --syntax-check

$ ansible-playbook p31-runcfg-bkup.yml

$ ls -l R*.txt
```

#### Step-3: Setup a cron job on the Ansible Controller to execute the playbook once a day.

- Execute the "date" command to get the current time on the server.

```
cisco@ansible-controller:~$ date
Tue Jan  8 23:27:59 UTC 2019

cisco@ansible-controller:~$ crontab -e
no crontab for cisco - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 2
```
- Inside the VIM editor of the crontab schedule the playbook to be run 1 or 2 mins after the current server time.
- Note: You must exit from "crontab -e" in order for the cron file to be installed.

```
#Run Ansible Playbook rtr-cfg-bkup everyday at 5:15 am UTC to backup router configs

15 5 * * * /usr/bin/ansible-playbook -i /etc/ansible/hosts  /home/cisco/p31-runcfg-bkup.yml

```
- Check to see if the cron job successfully created the new backup files.

$ ls -l R*.txt


### Conclusion
- You applied two modules, raw and copy, to retrieve and save the router config.
- You setup a cron job to automatically execute a playbook once a day.
- Review the section and discuss if you have any questions.

### Optional exercise

> - Add the below requirements to the above config backup playbook:
>    - Write the running config to startup config on IOS devices
>    - Gather the Admin mode running config from XR devices and save it to a file
> - Execute your playbook and verify if the results meet the requirements.
> - Solution playbook files (op31-runcfg-bkup.yml) are given in the appendix section 4.4 for your reference.

### Reference

> - Copy module: http://docs.ansible.com/ansible/latest/modules/copy_module.html
> - inventory_hostname: http://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html

### Example output

```
cisco@ansible-controller:~$ cat p31-runcfg-bkup.yml
---
- name: backup all routers config
  hosts: all

  tasks:
    - name: Collect Show run from all routers
      raw: "show run"

      register: RUNCFG

    - set_fact: time="{{lookup('pipe','date \"+%Y-%m-%d-%H-%M\"')}}"

    - name: save output to a file
      connection: local
      copy:
        content="\n ===show run=== \n {{ RUNCFG.stdout }}"
        dest="./{{ inventory_hostname }}_run_cfg_{{ time }}.txt"


cisco@ansible-controller:~$ ansible-playbook p31-runcfg-bkup.yml --syntax-check

playbook: p31-runcfg-bkup.yml
cisco@ansible-controller:~$
cisco@ansible-controller:~$
cisco@ansible-controller:~$ ansible-playbook p31-runcfg-bkup.yml

PLAY [Get Router Config from All Routers] ***************************************************************************************************************

TASK [Collect Show run from all routers] ****************************************************************************************************************
changed: [R1]
changed: [R2]

TASK [set_fact] *****************************************************************************************************************************************
ok: [R1]
ok: [R2]

TASK [save output to a file] ****************************************************************************************************************************
changed: [R2]
changed: [R1]

PLAY RECAP **********************************************************************************************************************************************
R1                         : ok=3    changed=2    unreachable=0    failed=0   
R2                         : ok=3    changed=2    unreachable=0    failed=0   


cisco@ansible-controller:~$ ls -ltr R*

-rw-rw-r-- 1 cisco cisco 1973 Jun  5 18:34 R2_2018-06-05-18-34.txt
-rw-rw-r-- 1 cisco cisco 5018 Jun  5 18:34 R1_2018-06-05-18-34.txt

```

---
## 3.2 Device health monitoring

### Objective

- Create a playbook to collect critical health data, process the data, and report issues.

### Approach

- Collect critical data by using iosxr_command module
- Use conditional functions to process the collected data and report any issues.
- Note: The playbook shown here is only applicable for XR devices. It can be modified for any other device by using the appropriate Ansible module.

### Lab exercise

#### Step-1: Create a playbook, named, p32-xr-health-monitoring.yml
- The first task will use the xr_command module to capture show command outputs from R2-XRv router.
- The second task will use the copy module to write the captured data into a file on the server. The inventory_hostname variable used here is a global variable defined in the inventory file, the output of inventory_hostname will be R2.
- The tasks under the debug module use a conditional statements to check the captured data for abnormalities.

```
cisco@ansible-controller:~$ vi p32-xr-health-monitoring.yml
---
- name: XR Router Health Monitoring
  hosts: XR
  gather_facts: false
  connection: local

  tasks:
    - name: Router Health Monitoring Commands
      iosxr_command:
        commands:
          - "{{item}}"

      with_items:
          - show platform
          - show redundancy
          - show proc cpu | ex "0%      0%       0%"
          - show memory sum location all | in "node|Pyhsical|available"
          - show ipv4 vrf all int bri
          - show route sum
          - show ospf neighbor
          - show mpls ldp neighbor | in "Id|Up|State"
          - show bgp all all sum | in "Address|^[0-9]+"

      register: iosxr_mon

    - name: Save output to a file
      copy:
        content="\n\n ===show platform=== \n\n {{ iosxr_mon.results[0].stdout_lines }} \n\n ===show redundancy=== \n\n {{ iosxr_mon.results[1].stdout }} \n\n ===show proc cpu=== \n\n {{ iosxr_mon.results[2].stdout }} \n\n ===show memory summary=== \n\n {{ iosxr_mon.results[3].stdout }} \n\n ===show ipv4 vrf all int bri=== \n\n {{ iosxr_mon.results[4].stdout }} \n\n ===show route sum=== \n\n {{ iosxr_mon.results[5].stdout }} \n\n ===show ospf nei=== \n\n {{ iosxr_mon.results[6].stdout }} \n\n ===show mpls ldp neighbor=== \n\n {{ iosxr_mon.results[7].stdout }} \n\n ===show bgp sum=== \n\n {{iosxr_mon.results[8].stdout }}"
        dest="./{{ inventory_hostname }}_health_check.txt"

    - name: Platform Hardware Check
      debug:
        msg: " {{ inventory_hostname }} show_platform indicates card is down"
      when: '"Down" in iosxr_mon.results[0].stdout[0]'

    - name: Redundancy Check
      debug:
        msg: " {{ inventory_hostname }} show_redundancy indicates card is not present"
      when: '"NSR not ready since Standby is not Present" in iosxr_mon.results[1].stdout[0]'

    - name: CPU Utilization Check
      debug:
         msg:
           - "{{ inventory_hostname }} CPU Utilization "
           - "{{ iosxr_mon.results[2].stdout_lines }}"

    - name: Memory Utilization Check
      debug:
        msg:
          - "{{ inventory_hostname }} Memory Utilization: "
          - "{{ iosxr_mon.results[3].stdout_lines }}"

    - name: Down Interface Checks
      debug:
        msg: " {{ inventory_hostname }} Interface is Down"
      when: '"Down" in iosxr_mon.results[4].stdout[0]'

    - name: Route Summary Check
      debug:
        msg: " {{ item }}"
      with_items:
        - " {{ inventory_hostname }} Route Summary: "
        - "{{ iosxr_mon.results[5].stdout_lines[0] }}"

    - name: OSPF Neighbor Check
      debug:
        msg:
          - " {{ inventory_hostname }} OSPF Neighbor Summary: "
          - " {{ iosxr_mon.results[6].stdout_lines }}"
      when: '"OSPF" in iosxr_mon.results[6].stdout[0]'

    - name: MPLS Neighbor Check
      debug:
        msg:
          - " {{ inventory_hostname }} MPLS LDP Summary: "
          - " {{ iosxr_mon.results[7].stdout_lines }}"
      when: '"Id" in iosxr_mon.results[7].stdout[0]'

    - name: BGP Neighbor Check
      debug:
        msg:
          - " {{ inventory_hostname }} BGP Sessions Down: "
          - " {{ iosxr_mon.results[8].stdout_lines }} "
      when: '"Active" in iosxr_mon.results[8].stdout[0]'
```

#### Step-2: Run the playbook

```
$ ansible-playbook p32-xr-health-monitoring.yml --syntax-check

$ ansible-playbook p32-xr-health-monitoring.yml
```

### Conclusion

- Ansible playbooks can be leveraged for Proactive health monitoring to identify issues
- Operations can proactively identify faults by periodically running the playbook in network
- Data collected can be used in-line or offline for analysis


### Example output

```
cisco@ansible-controller:~$ ansible-playbook p32-xr-health-monitoring.yml --syntax-check

playbook: p32-xr-health-monitoring.yml

cisco@ansible-controller:~$ ansible-playbook p32-xr-health-monitoring.yml

PLAY [XR Router Health Monitoring] *******************************************************************************************************************************************************************

TASK [Router Health Monitoring Commands] *************************************************************************************************************************************************************
ok: [R2] => (item=show platform)
ok: [R2] => (item=show redundancy)
ok: [R2] => (item=show proc cpu | ex "0%      0%       0%")
ok: [R2] => (item=show memory sum location all | in "node|Pyhsical|available")
ok: [R2] => (item=show ipv4 vrf all int bri)
ok: [R2] => (item=show route sum)
ok: [R2] => (item=show ospf neighbor)
ok: [R2] => (item=show mpls ldp neighbor | in "Id|Up|State")
ok: [R2] => (item=show bgp all all sum | in "Address|^[0-9]+")

TASK [Save output to a file] *************************************************************************************************************************************************************************
changed: [R2]

TASK [Platform Hardware Check] ***********************************************************************************************************************************************************************
skipping: [R2]

TASK [Redundancy Check] ******************************************************************************************************************************************************************************
ok: [R2] => {
    "msg": " R2 show_redundancy indicates card is not present"
}

TASK [CPU Utilization Check] *************************************************************************************************************************************************************************
ok: [R2] => {
    "msg": [
        "R2 CPU Utilization ",
        [
            [
                "CPU utilization for one minute: 1%; five minutes: 1%; fifteen minutes: 1%",
                " ",
                "PID    1Min    5Min    15Min Process"
            ]
        ]
    ]
}

TASK [Memory Utilization Check] **********************************************************************************************************************************************************************
ok: [R2] => {
    "msg": [
        "R2 Memory Utilization: ",
        [
            [
                "node:      node0_0_CPU0",
                "\fPhysical Memory: 3071M total (1443M available)",
                " Application Memory : 2868M (1443M available)"
            ]
        ]
    ]
}

TASK [Down Interface Checks] *************************************************************************************************************************************************************************
skipping: [R2]

TASK [Route Summary Check] ***************************************************************************************************************************************************************************
ok: [R2] => (item= R2 Route Summary: ) => {
    "msg": "  R2 Route Summary: "
}
ok: [R2] => (item=Route Source                     Routes     Backup     Deleted     Memory(bytes)) => {
    "msg": " Route Source                     Routes     Backup     Deleted     Memory(bytes)"
}
ok: [R2] => (item=connected                        1          1          0           320          ) => {
    "msg": " connected                        1          1          0           320          "
}
ok: [R2] => (item=local                            2          0          0           320          ) => {
    "msg": " local                            2          0          0           320          "
}
ok: [R2] => (item=dagr                             0          0          0           0            ) => {
    "msg": " dagr                             0          0          0           0            "
}
ok: [R2] => (item=Total                            3          1          0           640) => {
    "msg": " Total                            3          1          0           640"
}

TASK [OSPF Neighbor Check] ***************************************************************************************************************************************************************************
skipping: [R2]

TASK [MPLS Neighbor Check] ***************************************************************************************************************************************************************************
skipping: [R2]

TASK [BGP Neighbor Check] ****************************************************************************************************************************************************************************
skipping: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R2                         : ok=6    changed=1    unreachable=0    failed=0   
```

---

## 3.3 Method of Procedure (MOP) Automation

- MOP is a documented step-by-step sequence of tasks executed on network devices to achieve a planned objective. In this case, that objective is provisioning of OSPF.

### Objective
- Configure OSPF on both IOS and XR routers and enable OSPF on the loopback and GigE connected interface.
- Do not configure if OSPF is already found in the router.

### Steps Overview
- Step-1 : Create playbook to capture ospf data and run it as a pre-check
- Step-2: Configure OSPF on both routers
- Step-3: Run capture playbook again as post-check
- Step-4: Create one playbook to run through the full MOP

### Lab exercise

#### Step-1: OSPF data capture
- Create a playbook to capture OSPF data on both routers.
- Create a playbook called p33-ospf-capture.yml
- This playbook has two plays:
  - first one to capture and save OSPF data from IOS routers
  - second one to capture and save OSPF data from XR routers.

```
cisco@ansible-controller:~$ vi p33-ospf-capture.yml
---
- name: OSPF captures from IOS routers
  hosts: IOS
  connection: local

  tasks:
    - name: Collect IOS OSPF commands
      ios_command:
        authorize: yes
        commands:
           - show run | section ospf
           - show ip ospf interface brief
           - show ip ospf neighbor
           - show ip route ospf
      tags: always

      register: IOSPF

    - name: Save IOS precheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ IOSPF.stdout[0] }} \n\n ===show ip ospf int bri=== \n\n {{ IOSPF.stdout[1] }} \n\n ===show ip ospf nei=== \n\n {{ IOSPF.stdout[2] }} \n\n ===show ip route ospf=== \n\n {{ IOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Precheck.txt"
      tags: precheck

    - name: Save IOS postcheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ IOSPF.stdout[0] }} \n\n ===show ip ospf int bri=== \n\n {{ IOSPF.stdout[1] }} \n\n ===show ip ospf nei=== \n\n {{ IOSPF.stdout[2] }} \n\n ===show ip route ospf=== \n\n {{ IOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Postcheck.txt"
      tags: postcheck

- name: OSPF captures from XR routers
  hosts: XR
  connection: local

  tasks:
    - name: Collect XR OSPF commands
      iosxr_command:
        commands:
           - show run router ospf
           - show ospf interface brief
           - show ospf neighbor
           - show route ospf
      tags: always

      register: XOSPF

    - name: Save XR precheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ XOSPF.stdout[0] }} \n\n ===show ospf int bri=== \n\n {{ XOSPF.stdout[1] }} \n\n ===show ospf nei=== \n\n {{ XOSPF.stdout[2] }} \n\n===show route ospf=== \n\n {{ XOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Precheck.txt"
      tags: precheck

    - name: Save XR postcheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ XOSPF.stdout[0] }} \n\n ===show ospf int bri=== \n\n {{ XOSPF.stdout[1] }} \n\n ===show ospf nei=== \n\n {{ XOSPF.stdout[2] }} \n\n===show route ospf=== \n\n {{ XOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Postcheck.txt"
      tags: postcheck
```
- Predict the outcome of this playbook.
- Run the playbook. Note the **tags** option, that allows for section of the playbook to be called.
- We want the files to be saved as precheck since this capture is taken before configuring OSPF.
- Ensure that the playbook runs successfully and verify the existence of the precheck files in the home/cisco dir.

```
$ ansible-playbook p33-ospf-capture.yml --syntax-check

$ ansible-playbook p33-ospf-capture.yml --tags precheck

$ ls -l R[1-2]-OSPF*.txt
```

#### Step-2: Configure OSPF
- Create a playbook with 2 plays.
  - Play-1 will be to configure OSPF on IOS Routers
  - Play-2 will be to configure OSPF on XR Routers

- One of the requirements as outlined in the objectives above is not to configure OSPF if it is already present.
  - We achieved this using the "meta" module. Meta module is a special module which can directly influence the Ansible internal execution state.
  - Here, we search for the string, OSPF, and abort the play if it found.

- Create playbook, p33-ospf-config.yml, with the below contents:

```
cisco@ansible-controller:~$ vi p33-ospf-config.yml
---
- name: configure ospf on IOS routers - play-1
  hosts: IOS
  connection: local

  tasks:
    - name: pre-check for ospf config
      ios_command:
        commands:
          - show run | section ospf

      register: IOS_OSPF

    - meta: end_play
      when: '"router ospf" in IOS_OSPF.stdout[0]'

    - name: configure IOS ospf
      ios_config:
          parents: "router ospf 1"
          lines:
            - "router-id 192.168.0.1"
            - "passive-interface Loopback0"
            - "network 192.168.0.1 0.0.0.0 area 0"
            - "network 10.0.0.4 0.0.0.3 area 0"
          save_when: always

- name: configure ospf on XR routers - play-2
  hosts: XR
  connection: local

  tasks:
    - name: pre-check for ospf config
      iosxr_command:
        commands:
          - show run router ospf

      register: XR_OSPF

    - meta: end_play
      when: '"router ospf" in XR_OSPF.stdout[0]'

    - name: configure XR ospf
      iosxr_config:
        parents: "router ospf 1"
        lines:
          - "router-id 192.168.0.2"
          - "area 0"
          - "interface Loopback0"
          - "passive enable"
          - "exit"
          - "interface GigabitEthernet0/0/0/0"
          - "exit"
```
- Check the existence of OSPF on the routers

```
$ ansible IOS -m raw -a "sho run | sec ospf"

$ ansible XR -m raw -a "sho run router ospf"

$ ansible XR -m raw -a "show route ospf"
```

- If there is ospf config, you may delete it, or simply continue.

#### Step-3: Run the playbook and verify that OSPF route exist on the routers.

```
$ ansible-playbook p33-ospf-config.yml --syntax-check

$ ansible-playbook p33-ospf-config.yml

$ ansible IOS -m raw -a "show ip ospf neigh"

$ ansible IOS -m raw -a "show ip route ospf"

$ ansible XR -m raw -a "show ospf neigh"

$ ansible XR -m raw -a "show route ospf"
```

#### Step-4: Post-config data capture

- Run p33-ospf-capture.yml to collect the post capture data for OSPF on both routers.
- Review the playbook above, p33-ospf-capture.yml. Look for `tags: postcheck`
- Predict the outcome of this playbook, with tags=postcheck.
- Run the playbook:
- Ensure that the playbook runs successfully and verify the existence of the post-check files in the home dir, /home/cisco

```
$ ansible-playbook p33-ospf-capture.yml --syntax-check

$ ansible-playbook p33-ospf-capture.yml --tags postcheck

$ ls -l R[1-2]-OSPF*.txt
```

#### Step-5: Create one playbook to run through the full MOP
- Create a playbook, named p33-ospf-mop.yml, with the contents below:
- Use the pause module to insert a delay in ansible execution to allow for ospf sessions to come up before conducting postchecks.

```
cisco@ansible-controller:~$ vi p33-ospf-mop.yml
---
- name: pre-config captures
  hosts: localhost

  tasks:
    - name: run precheck playbook
      command: ansible-playbook p33-ospf-capture.yml --tags precheck

- name: configure OSPF on both routers
  import_playbook: p33-ospf-config.yml

- name: post-config captures
  hosts: localhost

  tasks:
    - pause: seconds=30

    - name: run postcheck playbook
      command: ansible-playbook p33-ospf-capture.yml --tags postcheck
```

- Predict the outcome of this playbook
- Run the playbook:

```
$ ansible-playbook p33-ospf-mop.yml --syntax-check

$ ansible-playbook p33-ospf-mop.yml -v

$ ls -l R[1-2]-OSPF*.txt
```

### Conclusion
- Production MOPs require more stringent checks than shown here. This was a basic exercise to show how the 3 phases: Pre-check, Config Change, and Post-Check could be done using Ansible.
- Review the section and discuss if you have any questions.

### Optional exercise

> - Add the below requirements to the MOP playbook:
>    - Create a new playbook that compares the differences between the pre-capture and post-capture files.
>    - Hint: Use the command module to call the Linux diff utility.
> - Execute your playbook and verify if the results meet the requirements.
> - Solution playbook files (op33-diff.yml) are given in the appendix section 4.5 for your reference.


### Reference

> - copy module: http://docs.ansible.com/ansible/latest/modules/copy_module.html
> - meta module: http://docs.ansible.com/ansible/latest/modules/meta_module.html
> - pause module: http://docs.ansible.com/ansible/latest/modules/pause_module.html


### Example output
```
cisco@ansible-controller:~$ vi p33-ospf-capture.yml
---
- name: OSPF captures from IOS routers
  hosts: IOS
  connection: local

  tasks:
    - name: Collect IOS OSPF commands
      ios_command:
        authorize: yes
        commands:
           - show run | section ospf
           - show ip ospf interface brief
           - show ip ospf neighbor
           - show ip route ospf
      tags: always

      register: IOSPF

    - name: Save IOS precheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ IOSPF.stdout[0] }} \n\n ===show ip ospf int bri=== \n\n {{ IOSPF.stdout[1] }} \n\n ===show ip ospf nei=== \n\n {{ IOSPF.stdout[2] }} \n\n ===show ip route ospf=== \n\n {{ IOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Precheck.txt"
      tags: precheck

    - name: Save IOS postcheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ IOSPF.stdout[0] }} \n\n ===show ip ospf int bri=== \n\n {{ IOSPF.stdout[1] }} \n\n ===show ip ospf nei=== \n\n {{ IOSPF.stdout[2] }} \n\n ===show ip route ospf=== \n\n {{ IOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Postcheck.txt"
      tags: postcheck

- name: OSPF captures from XR routers
  hosts: XR
  connection: local

  tasks:
    - name: Collect XR OSPF commands
      iosxr_command:
        commands:
           - show run router ospf
           - show ospf interface brief
           - show ospf neighbor
           - show route ospf
      tags: always

      register: XOSPF

    - name: Save XR precheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ XOSPF.stdout[0] }} \n\n ===show ospf int bri=== \n\n {{ XOSPF.stdout[1] }} \n\n ===show ospf nei=== \n\n {{ XOSPF.stdout[2] }} \n\n===show route ospf=== \n\n {{ XOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Precheck.txt"
      tags: precheck

    - name: Save XR postcheck output to a file
      copy:
        content=" \n\n ===show run router ospf=== \n\n {{ XOSPF.stdout[0] }} \n\n ===show ospf int bri=== \n\n {{ XOSPF.stdout[1] }} \n\n ===show ospf nei=== \n\n {{ XOSPF.stdout[2] }} \n\n===show route ospf=== \n\n {{ XOSPF.stdout[3] \n\n }} "
        dest="./{{ inventory_hostname }}-OSPF-Postcheck.txt"
      tags: postcheck

cisco@ansible-controller:~$ ansible-playbook p33-ospf-capture.yml --syntax-check

playbook: p33-ospf-capture.yml

cisco@ansible-controller:~$ ansible-playbook p33-ospf-capture.yml --tags precheck

PLAY [OSPF captures from IOS routers] ****************************************************************************************************************************************************************

TASK [Collect IOS OSPF commands] *********************************************************************************************************************************************************************
ok: [R1]

TASK [Save IOS precheck output to a file] ************************************************************************************************************************************************************
changed: [R1]

PLAY [OSPF captures from XR routers] *****************************************************************************************************************************************************************

TASK [Collect XR OSPF commands] **********************************************************************************************************************************************************************
ok: [R2]

TASK [Save XR precheck output to a file] *************************************************************************************************************************************************************
changed: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=1    unreachable=0    failed=0   
R2                         : ok=2    changed=1    unreachable=0    failed=0   


cisco@ansible-controller:~$ ls -l R[1-2]-OSPF*.txt
-rw-rw-r-- 1 cisco cisco 145 Jan  9 00:15 R1-OSPF-Precheck.txt
-rw-rw-r-- 1 cisco cisco 178 Jan  9 00:16 R2-OSPF-Precheck.txt

cisco@ansible-controller:~$ vi p33-ospf-config.yml
---
- name: configure ospf on IOS routers - play-1
  hosts: IOS
  connection: local

  tasks:
    - name: pre-check for ospf config
      ios_command:
        commands:
          - show run | section ospf

      register: IOS_OSPF

    - meta: end_play
      when: '"router ospf" in IOS_OSPF.stdout[0]'

    - name: configure IOS ospf
      ios_config:
          parents: "router ospf 1"
          lines:
            - "router-id 192.168.0.1"
            - "passive-interface Loopback0"
            - "network 192.168.0.1 0.0.0.0 area 0"
            - "network 10.0.0.4 0.0.0.3 area 0"
          save_when: always

- name: configure ospf on XR routers - play-2
  hosts: XR
  connection: local

  tasks:
    - name: pre-check for ospf config
      iosxr_command:
        commands:
          - show run router ospf

      register: XR_OSPF

    - meta: end_play
      when: '"router ospf" in XR_OSPF.stdout[0]'

    - name: configure XR ospf
      iosxr_config:
        parents: "router ospf 1"
        lines:
          - "router-id 192.168.0.2"
          - "area 0"
          - "interface Loopback0"
          - "passive enable"
          - "exit"
          - "interface GigabitEthernet0/0/0/0"
          - "exit"

cisco@ansible-controller:~$ ansible IOS -m raw -a "sho run | sec ospf"
R1 | CHANGED | rc=0 >>
 ip ospf cost 1
Shared connection to 172.16.101.191 closed.

cisco@ansible-controller:~$ ansible XR -m raw -a "sho run router ospf"
R2 | CHANGED | rc=0 >>

Wed Jan  9 00:19:30.891 UTC
% No such configuration item(s)

Shared connection to 172.16.101.192 closed.

cisco@ansible-controller:~$ ansible XR -m raw -a "show route ospf"
R2 | CHANGED | rc=0 >>

Wed Jan  9 00:19:41.770 UTC

% No matching routes found

Shared connection to 172.16.101.192 closed.

cisco@ansible-controller:~$ ansible-playbook p33-ospf-config.yml --syntax-check

playbook: p33-ospf-config.yml

cisco@ansible-controller:~$ ansible-playbook p33-ospf-config.yml

PLAY [configure ospf on IOS routers - play-1] ********************************************************************************************************************************************************

TASK [pre-check for ospf config] *********************************************************************************************************************************************************************
ok: [R1]

TASK [configure IOS ospf] ****************************************************************************************************************************************************************************
changed: [R1]

PLAY [configure ospf on XR routers - play-2] *********************************************************************************************************************************************************

TASK [pre-check for ospf config] *********************************************************************************************************************************************************************
ok: [R2]

TASK [configure XR ospf] *****************************************************************************************************************************************************************************
changed: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=1    unreachable=0    failed=0   
R2                         : ok=2    changed=1    unreachable=0    failed=0   

cisco@ansible-controller:~$ ansible IOS -m raw -a "show ip ospf neigh"
R1 | CHANGED | rc=0 >>

Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.0.2       1   2WAY/DROTHER    00:00:36    10.0.0.6        GigabitEthernet2Shared connection to 172.16.101.191 closed.


cisco@ansible-controller:~$ ansible IOS -m raw -a "show ip ospf neigh"
R1 | CHANGED | rc=0 >>

Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.0.2       1   FULL/DR         00:00:32    10.0.0.6        GigabitEthernet2Shared connection to 172.16.101.191 closed.


cisco@ansible-controller:~$ ansible IOS -m raw -a "show ip route ospf"
R1 | CHANGED | rc=0 >>

Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      192.168.0.0/32 is subnetted, 2 subnets
O        192.168.0.2 [110/2] via 10.0.0.6, 00:00:35, GigabitEthernet2Shared connection to 172.16.101.191 closed.


cisco@ansible-controller:~$ ansible XR -m raw -a "show ospf neigh"
R2 | CHANGED | rc=0 >>

Wed Jan  9 00:22:24.789 UTC

* Indicates MADJ interface
# Indicates Neighbor awaiting BFD session up

Neighbors for OSPF 1

Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.0.1     1     FULL/BDR        00:00:30    10.0.0.5        GigabitEthernet0/0/0/0
    Neighbor is up for 00:01:17

Total neighbor count: 1

Shared connection to 172.16.101.192 closed.


cisco@ansible-controller:~$ ansible XR -m raw -a "show route ospf"
R2 | CHANGED | rc=0 >>

Wed Jan  9 00:22:30.898 UTC

O    192.168.0.1/32 [110/2] via 10.0.0.5, 00:00:49, GigabitEthernet0/0/0/0

Shared connection to 172.16.101.192 closed.

cisco@ansible-controller:~$ ansible-playbook p33-ospf-capture.yml --syntax-check

playbook: p33-ospf-capture.yml
cisco@ansible-controller:~$ ansible-playbook p33-ospf-capture.yml --tags postcheck

PLAY [OSPF captures from IOS routers] ****************************************************************************************************************************************************************

TASK [Collect IOS OSPF commands] *********************************************************************************************************************************************************************
ok: [R1]

TASK [Save IOS postcheck output to a file] ***********************************************************************************************************************************************************
changed: [R1]

PLAY [OSPF captures from XR routers] *****************************************************************************************************************************************************************

TASK [Collect XR OSPF commands] **********************************************************************************************************************************************************************
ok: [R2]

TASK [Save XR postcheck output to a file] ************************************************************************************************************************************************************
changed: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=1    unreachable=0    failed=0   
R2                         : ok=2    changed=1    unreachable=0    failed=0  

cisco@ansible-controller:~$ ls -l R[1-2]-OSPF*.txt
-rw-rw-r-- 1 cisco cisco 1410 Jan  9 00:23 R1-OSPF-Postcheck.txt
-rw-rw-r-- 1 cisco cisco  145 Jan  9 00:15 R1-OSPF-Precheck.txt
-rw-rw-r-- 1 cisco cisco  969 Jan  9 00:23 R2-OSPF-Postcheck.txt
-rw-rw-r-- 1 cisco cisco  178 Jan  9 00:16 R2-OSPF-Precheck.txt

cisco@ansible-controller:~$ ansible-playbook p33-ospf-mop.yml --syntax-check

playbook: p33-ospf-mop.yml

cisco@ansible-controller:~$ ansible-playbook p33-ospf-mop.yml

PLAY [pre-config captures] ***************************************************************************************************************************************************************************

TASK [run precheck playbook] *************************************************************************************************************************************************************************
changed: [localhost]

PLAY [configure ospf on IOS routers - play-1] ********************************************************************************************************************************************************

TASK [pre-check for ospf config] *********************************************************************************************************************************************************************
ok: [R1]

PLAY [configure ospf on XR routers - play-2] *********************************************************************************************************************************************************

TASK [pre-check for ospf config] *********************************************************************************************************************************************************************
ok: [R2]

PLAY [post-config captures] **************************************************************************************************************************************************************************

TASK [pause] *****************************************************************************************************************************************************************************************
Pausing for 30 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [localhost]

TASK [run postcheck playbook] ************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=1    changed=0    unreachable=0    failed=0   
R2                         : ok=1    changed=0    unreachable=0    failed=0   
localhost                  : ok=3    changed=2    unreachable=0    failed=0   

cisco@ansible-controller:~$ ls -l R[1-2]-OSPF*.txt
-rw-rw-r-- 1 cisco cisco 1410 Jan  9 00:28 R1-OSPF-Postcheck.txt
-rw-rw-r-- 1 cisco cisco 1410 Jan  9 00:27 R1-OSPF-Precheck.txt
-rw-rw-r-- 1 cisco cisco  969 Jan  9 00:28 R2-OSPF-Postcheck.txt
-rw-rw-r-- 1 cisco cisco  969 Jan  9 00:27 R2-OSPF-Precheck.txt

```

## 3.4 Generate Device Configuration

- Network configuration and rollouts are critical part of daily network operations.
- This exercise will simulate a network config generation using Ansible Roles and Jinj2 to develop config templates.

### Objectives

- Create a playbook to generate configuration based on Roles directory structures.
- You will gain an understanding of how Ansible Roles and Jinja2 Templates can be used to generate router configurations.

### Steps Overview

- Step-1: Manually create the roles directory structure for the 2 router types (IOS & XR).
- Step-2: Create a playbook roles-bgp.yml to call the two roles (ios-bgp & xr-bgp) to generate the iBGP config and to upload the generated config to the routers.
- Step-3: Create main.yml files under the tasks and vars folders to setup the tasks to be executed when the roles are called.
- Step-4: Create the iBGP configuration template for ios and xr router using Jinja2 templating format.
- Step-5: Execute the playbook roles-bgp.yml to generate the iBGP config and apply it to both routers.

### Lab Exercise

#### Step-1: Create two sub-folders by the name “ios-bgp”  and “xr-bgp” using mkdir command

```
cisco@ansible-controller:~$ pwd
/home/cisco

cisco@ansible-controller:~$ mkdir ios-bgp xr-bgp
```
- Verify the directories were created.
```
cisco@ansible-controller:~$ ls *bgp
ios-bgp:

xr-bgp:
```

#### Step-2: Create tasks, vars, and templates directories under both ios-bgp and xr-bgp folder.

```
cisco@ansible-controller:~$ mkdir ios-bgp/tasks ios-bgp/vars ios-bgp/templates

cisco@ansible-controller:~$ mkdir xr-bgp/tasks xr-bgp/vars xr-bgp/templates
```
- Verify the directory structure
```
cisco@ansible-controller:~$ tree ios-bgp
ios-bgp
├── tasks
├── templates
└── vars

3 directories, 0 files

cisco@ansible-controller:~$ tree xr-bgp
xr-bgp
├── tasks
├── templates
└── vars
```

#### Step-3: Create a playbook called roles-bgp.yml.
- This playbook contains 3 plays:
  - First play will run on the localhost (ansible-controller) and call on the roles to generate IOS and XR iBGP configs.
  - The second and third plays are used to upload the BGP configs to the routers.

```
cisco@ansible-controller:~$ vi roles-bgp.yml
---
- name: Generate router bgp configuration files using Roles and Jinja2 Templates
  hosts: localhost

  roles:
   - xr-bgp
   - ios-bgp

- name: Task to upload config to R1-IOS
  hosts: IOS
  gather_facts: false
  connection: local
  tasks:
  - name: "Load iBGP configs for R1 router using SRC option using IOS_CONFIG Module"
    ios_config:
      src: "/home/cisco/R1-CSR1K-BGP.txt"

- name: Task to upload config to R2-XRV
  gather_facts: false
  connection: local
  hosts: XR

  tasks:
  - name: "Load iBGP configs for R2 router using SRC option of IOSXR_CONFIG Module "
    iosxr_config:
      src: "/home/cisco/R2-XRv-BGP.txt"

```

#### Step-4: Create a main.yml file inside the ios-bgp/tasks folder.
- The main.yml file in the tasks sub-folder identifies the Jinja2 Template that contains the iBGP configuration template, specified under the template folder, for this role.

```
cisco@ansible-controller:~$ vi ios-bgp/tasks/main.yml
---
- name: Generate R1 IOS router iBGP config file
  template: src=IOS-BGP.j2 dest=./{{item.hostname}}-BGP.txt
  with_items: "{{router_list}}"
```

#### Step-5: Create a main.yml file inside the ios-bgp/vars folder.
- Vars file contains all the values for parameters given in the Jinj2 template file.

```
cisco@ansible-controller:~$ vi ios-bgp/vars/main.yml
---
router_list:
  -  hostname: R1-CSR1K
     profile: IOS-XE
     RID: 192.168.0.1
     PEER_IP: 192.168.0.2
     LCL_ASN: 1
     RMT_ASN: 1
```

#### Step-6: Create a IOS-BGP.j2 template file inside the ios-bgp/templates folder.

- Note vi users do the following to create the file.

```
cisco@ansible-controller:~$ vi ios-bgp/templates/IOS-BGP.j2

router bgp {{item.LCL_ASN}}
 bgp router-id {{item.RID}}
 bgp log-neighbor-changes
 neighbor {{item.PEER_IP}} remote-as {{item.RMT_ASN}}
 neighbor {{item.PEER_IP}} update-source Loopback0
!
```

#### Step-7: Same steps will need to be repeated for the xr-bgp role. Create a main.yml file inside the xr-bgp/tasks folder.

```
cisco@ansible-controller:~$ vi xr-bgp/tasks/main.yml
---
- name: Generate R2 XRV router iBGP config file
  template: src=XR-BGP.j2 dest=./{{item.hostname}}-BGP.txt
  with_items: "{{router_list}}"
```

#### Step-8: Create a main.yml file inside the xr-bgp/vars folder.

```
cisco@ansible-controller:~$ vi xr-bgp/vars/main.yml

router_list:
  -  hostname: R2-XRv
     profile: IOS-XR
     RID: 192.168.0.2
     PEER_IP: 192.168.0.1
     LCL_ASN: 1
     RMT_ASN: 1
```

#### Step-9: Create a XR-BGP.j2 file inside the xr-bgp/templates folder.

```
cisco@ansible-controller:~$ vi xr-bgp/templates/XR-BGP.j2

router bgp {{item.LCL_ASN}}
 bgp router-id {{item.RID}}
 address-family ipv4 unicast
 !
 neighbor {{item.PEER_IP}}
  remote-as {{item.RMT_ASN}}
  update-source Loopback0
  address-family ipv4 unicast
  !
 !
!
```

#### Step-10: Run the tree command and validate that following files structure is created

```
cisco@ansible-controller:~$ tree ios-bgp
ios-bgp
├── tasks
│   └── main.yml
├── templates
│   └── IOS-BGP.j2
└── vars
    └── main.yml

3 directories, 3 files
cisco@ansible-controller:~$ tree xr-bgp
xr-bgp
├── tasks
│   └── main.yml
├── templates
│   └── XR-BGP.j2
└── vars
    └── main.yml

3 directories, 3 files
```

#### Step-11: Execute the roles-bgp.yml playbook.

```
$ ansible-playbook roles-bgp.yml --syntax-check

$ ansible-playbook roles-bgp.yml
```

#### Step-12: After the playbook is run, there should be 2 files generated (R1-CSR1K-BGP.txt & R2-XRv-BGP.txt).

```
cisco@ansible-controller:~$ ls -l R[1-2]-*-BGP.txt
-rw-rw-r-- 1 cisco cisco 148 Jan  9 00:59 R1-CSR1K-BGP.txt
-rw-rw-r-- 1 cisco cisco 173 Jan  9 00:59 R2-XRv-BGP.txt

cisco@ansible-controller:~$ more R1-CSR1K-BGP.txt
router bgp 1
 bgp router-id 192.168.0.1
 bgp log-neighbor-changes
 neighbor 192.168.0.2 remote-as 1
 neighbor 192.168.0.2 update-source Loopback0
!
cisco@ansible-controller:~$ more R2-XRv-BGP.txt
router bgp 1
 bgp router-id 192.168.0.2
 address-family ipv4 unicast
 !
 neighbor 192.168.0.1
  remote-as 1
  update-source Loopback0
  address-family ipv4 unicast
  !
 !
!
```

#### Step-13: Make sure iBGP was configured on both routers.

```
cisco@ansible-controller:~$ ansible all -m raw -a "show bgp sum"
```

- Try rerunning the Health Monitoring Playbook on R2 to check state of OSPF and BGP routing protocols.
- Recall: BGP output will only be printed if BGP sessions are down.

```
cisco@ansible-controller:~$ ansible-playbook p32-xr-health-monitoring.yml
```

### Conclusion

- Ansible roles can be utilized to organize large playbooks.
- Roles creates a separation of functions: variables, tasks, and templates in unique directories.
- Ansible roles expect a main.yml file under the sub-directory.

### Example output:
```
cisco@ansible-controller:~$ ansible-playbook roles-bgp.yml --syntax-check

playbook: roles-bgp.yml
cisco@ansible-controller:~$ ansible-playbook roles-bgp.yml

PLAY [Generate router bgp configuration files using Roles and Jinja2 Templates] **********************************************************************************************************************

TASK [xr-bgp : Generate R2 XRV router iBGP config file] **********************************************************************************************************************************************
changed: [localhost] => (item={u'profile': u'IOS-XR', u'LCL_ASN': 1, u'RMT_ASN': 1, u'RID': u'192.168.0.2', u'hostname': u'R2-XRv', u'PEER_IP': u'192.168.0.1'})

TASK [ios-bgp : Generate R1 IOS router iBGP config file] *********************************************************************************************************************************************
changed: [localhost] => (item={u'profile': u'IOS-XE', u'LCL_ASN': 1, u'RMT_ASN': 1, u'RID': u'192.168.0.1', u'hostname': u'R1-CSR1K', u'PEER_IP': u'192.168.0.2'})

PLAY [Task to upload config to R1-IOS] ******************************************************************************************************************************************************************

TASK [Load iBGP configs for R1 router using SRC option using IOS_CONFIG Module] **********************************************************************************************************************
changed: [R1]

PLAY [Task to upload config to R2-XRV] ***************************************************************************************************************************************************************

TASK [Load iBGP configs for R2 router using SRC option of IOSXR_CONFIG Module] ***********************************************************************************************************************
changed: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=1    changed=1    unreachable=0    failed=0   
R2                         : ok=1    changed=1    unreachable=0    failed=0   
localhost                  : ok=2    changed=2    unreachable=0    failed=0   

cisco@ansible-controller:~$ ls -l R[1-2]-*-BGP.txt
-rw-rw-r-- 1 cisco cisco 148 Jan  9 00:59 R1-CSR1K-BGP.txt
-rw-rw-r-- 1 cisco cisco 173 Jan  9 00:59 R2-XRv-BGP.txt

```
---

## 3.5 Bulk Config Generation

- By Leveraging Ansible Roles and templates, users can build bulk configurations for deployment at scale.

### Objective

- Build configurations for 2 different type of router OS: IOS and XR.

### Steps Overview

- Initialize the roles directory and file structure by using ansible-galaxy cli
- Build the playbook, template and variables for bulk config generation

### Lab Exercise

#### Step-1: Create a new role called config-gen by using the ansible-galaxy utility.

- Note: Ansible-Galaxy command is built into Ansible and allows for an automated way to create the Roles directory structure.

```
cisco@ansible-controller:~$ pwd
/home/cisco

cisco@ansible-controller:~$ ansible-galaxy init config-gen
- config-gen was created successfully
```
#### Step-2: Review the tree structure that has been created by galaxy.

- For this lab, you will be using the templates, vars, and tasks sub-directories to generate the bulk configuration.

```
cisco@ansible-controller:~$ tree config-gen/
config-gen/
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```
#### Step-3: Edit the main.yml file under the config-gen/tasks sub-directory.

```
$ vi config-gen/tasks/main.yml
---
- name: Generate the configuration for xr-routers
  template:
     src=xr-config-template.j2
     dest=./{{item.hostname}}.txt
  with_items:
     - "{{ xr_hostnames }}"

- name: Generate the configuration for ios-routers
  template:
     src=ios-config-template.j2
     dest=./{{item.hostname}}.txt
  with_items:
     - "{{ ios_hostnames }}"
# tasks file for config-gen
```

#### Step-4: Create the platform specific configuration and save them under the config-gen/templates folder.

- 4a. Create the following template “xr-config-template.j2” under the templates directory.

```
cisco@ansible-controller:~$ vi config-gen/templates/xr-config-template.j2

hostname {{item.hostname}}
service timestamps log datetime msec
service timestamps debug datetime msec
clock timezone {{item.timezone}} {{item.timezone_offset}}
clock summer-time {{item.timezone_dst}} recurring
telnet vrf default ipv4 server max-servers 10
telnet vrf Mgmt-intf ipv4 server max-servers 10
domain lookup disable
vrf Mgmt-intf
 address-family ipv4 unicast
 !
 address-family ipv6 unicast
 !
!
domain name virl.info
ssh server v2
ssh server vrf Mgmt-intf
!
line template vty
timestamp
exec-timeout 720 0
!
line console
exec-timeout 0 0
!
line default
exec-timeout 720 0
!
vty-pool default 0 50
control-plane
 management-plane
  inband
   interface all
    allow all
   !
  !
 !
!
!
cdp
!
!
interface Loopback0
  description Loopback
  ipv4 address {{item.loopback0_ip}} {{item.loopback0_mask}}
!
interface GigabitEthernet0/0/0/0
  description to R1-CSR1kv
  ipv4 address {{item.gig0000_ip}} {{item.gig0000_mask}}
  cdp
  no shutdown
!
interface GigabitEthernet0/0/0/1
  description to R3-NXOS
  ipv4 address {{item.gig0001_ip}} {{item.gig0001_mask}}
  cdp
  no shutdown
!
router ospf 16509
  log adjacency changes
  router-id {{item.loopback0_ip}}
  address-family ipv4
  area 0
    !
    interface Loopback0
      passive enable
    !
{% for interface in xr_interfaces %}
interface {{interface}}
cost 1
!
{% endfor %}
  !
!
!

```

- 4b. Create the following template “ios-config-template.j2” under the templates directory.

```
cisco@ansible-controller:~$ vi config-gen/templates/ios-config-template.j2

clock timezone {{item.timezone}} {{item.timezone_offset}}
clock summer-time {{item.timezone_dst}} recurring

service timestamps debug datetime msec
service timestamps log datetime msec
platform qfp utilization monitor load 80
no platform punt-keepalive disable-kernel-core
platform console serial
!
hostname {{item.hostname}}
!
boot-start-marker
boot-end-marker
!
!
vrf definition Mgmt-intf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
enable secret 4 tnhtc92DXBhelxjYk8LWJrPV36S2i4ntXrpb4RFmfqY
enable password cisco
!
no aaa new-model
!
!
!
!
!
!
!
!

no ip domain lookup
ip domain name virl.info
!
!
!
ipv6 unicast-routing
!
!
!
!
!
!
!
subscriber templating
!
!
!
!
!
!
!
multilink bundle-name authenticated
!
!
!
!
!
crypto pki trustpoint TP-self-signed-35466579
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-35466579
 revocation-check none
 rsakeypair TP-self-signed-35466579
!
!
crypto pki certificate chain TP-self-signed-35466579
 certificate self-signed 01
  3082032C 30820214 A0030201 02020101 300D0609 2A864886 F70D0101 05050030
  2F312D30 2B060355 04031324 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 33353436 36353739 301E170D 31383033 32383136 33343532
  5A170D32 30303130 31303030 3030305A 302F312D 302B0603 55040313 24494F53
  2D53656C 662D5369 676E6564 2D436572 74696669 63617465 2D333534 36363537
  39308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201 0A028201
  0100C122 3C95D116 714EE581 53539DCE 33BBE636 20BCAB70 B12ECDE8 832DB71C
  F223B066 E3779F87 0BF81EE6 CE6E60EE F471B22F 5ECE57FD 50C7D706 17F3F62D
  4573882F B9B6351F ECDC6192 167D768A DC8B4613 8A2AEB70 1906E49D 0A2734A8
  64C0C7A3 4B6951D2 573AFF96 5682BE7D 305F4351 A5E6A667 DB787283 724AF55F
  3A049F98 57A1C34F CC9B9C24 3056B3DF 11A04AB4 3F051C0D 14D5AACE B7B0D991
  611FE0D6 6B2CC9D2 3F410224 52701D25 135C7BF2 FEDC0BCD F9BD7C10 4B437143
  E38A10E8 F5423F0E BB71A593 AFDBC814 D6DD4ED6 0709FCC5 33F480F0 6389C2AF
  F0C36163 54164A20 541AAA30 EAFDFD2B 35361640 82331C9B F0D97302 B1429508
  87DB0203 010001A3 53305130 0F060355 1D130101 FF040530 030101FF 301F0603
  551D2304 18301680 14B0C21C 0050185E 5D0751E6 6A90DD48 D9157E6B 0E301D06
  03551D0E 04160414 B0C21C00 50185E5D 0751E66A 90DD48D9 157E6B0E 300D0609
  2A864886 F70D0101 05050003 82010100 4E89908F 13A8518B 33D0DC0E 71548510
  7E3285F7 71E4B8A4 2E25FA83 3FD571F5 17D190EA DFC4F076 AF1C3494 17DC54B4
  93A61630 C2D321BE F3D1B9D1 72AA7BE9 D5755FD5 C2330B82 F9DA1B4B 590BBA8A
  0A36758E 22061021 86D03C8B D5877680 954F22E6 3A4F807E 79CA5DB5 F63ECF74
  CA45C80D A8052A3A 48CD69B5 027D66D5 08020FA6 94FCE404 07D12573 590C0D60
  5999C40B FECA7B2D A11FC2B8 21D7A110 E4814E8E 2ED74D9D B22A66DF B9BF8932
  424A5807 AF9A59B5 FB6A7FCE B73E25B8 F937695D 9E15768D 614AA387 0B26B6FA
  C54DF6E2 34E5E803 1123AB24 9CC8F3CF FDBB6B7E CC3FF86C B83C858A 34646F0B
  0C79ED3D 814ACA2F 3F565B5C BB84FCAA
  	quit
!
!
!
!
!
!
!
!
license udi pid CSR1000V sn 9N7CZX65NJ3
license accept end user agreement
license boot level ax
diagnostic bootup level minimal
!
spanning-tree extend system-id
!
!
username cisco privilege 15 secret 5 $1$F6GC$L/.gqoiPm0AcItLajjXXJ/
!
redundancy
!
!
cdp run
!

!
interface Loopback0
 description Loopback
 ip address {{item.loopback0_ip}} {{item.loopback0_mask}}
!
interface GigabitEthernet2
 description to R2-XRv
 ip address {{item.gigaethernet1_ip}} {{item.gigaethernet1_mask}}
 ip ospf cost 1
 negotiation auto
 cdp enable
 no mop enabled
 no mop sysid
!
!
router ospf 16509
  network {{item.ospf_network1}} {{item.ospf_network_mask1}} area {{item.areaid}}
  log-adjacency-changes
  passive-interface Loopback0
  network {{item.ospf_network2}} {{item.ospf_network_mask1}} area {{item.areaid}}
  network {{item.ospf_network3}} {{item.ospf_network_mask1}} area {{item.areaid}}
!
!


threat-visibility
!
virtual-service csr_mgmt
!
ip forward-protocol nd
ip http server
ip http authentication local
ip http secure-server
!
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh server algorithm authentication password
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
!
control-plane
!
!
line con 0
 password cisco
 stopbits 1
line vty 0 4
 exec-timeout 720 0
 password cisco
 login local
 transport input telnet ssh
!
end
```

#### Step-5: Define the variable needed to generate the config in the main.yml file under the config-gen/vars sub-directory.

```
cisco@ansible-controller:~$ vi config-gen/vars/main.yml
---
xr_hostnames:
   - { hostname: xr-router-rtr1, timezone: EST, timezone_dst: EDT, timezone_offset: -5, loopback0_ip: 192.168.0.1, loopback0_mask: 255.255.255.255, gig0000_ip: 10.0.0.5, gig0000_mask: 255.255.255.0, gig0001_ip: 10.1.0.5 , gig0001_mask: 255.255.255.0,}

xr_interfaces:
  - GigabitEthernet0/0/0/0
  - GigabitEthernet0/0/0/1


ios_hostnames:
   - { hostname: ios-router-rtr1, timezone: EST, timezone_dst: EDT, timezone_offset: -5, loopback0_ip: 192.168.0.2, loopback0_mask: 255.255.255.255, gigaethernet1_ip: 10.0.0.6, gigaethernet1_mask: 255.255.255.0, ospf_network1: 192.168.0.2, ospf_network_mask1: 0.0.0.0, ospf_network2: 10.0.0.0, ospf_network_mask2: 0.0.0.255, ospf_network3: 10.1.0.0, ospf_network_mask3: 0.0.0.255, areaid: 0 }
# vars file for config-gen
```

#### Step-6: Create a playbook config-gen.yml to invoke the role of Bulk config generation. The config-gen.yml playbook has to be in the same directory as the config-gen role.

```
cisco@ansible-controller:~$ pwd
/home/cisco

cisco@ansible-controller:~$ vi config-gen.yml
---
  - name: Playbook to generate configuration based on role "config-gen"
    hosts: localhost

    roles:
       - config-gen
```

#### Step-7: Execute the playbook config-gen.yml. You will see the config files are generated in target location.

```
cisco@ansible-controller:~$ ansible-playbook config-gen.yml

```
#### Step-8: Validate the files are created
```
cisco@ansible-controller:~$ ls -l *rtr1*
-rw-rw-r-- 1 cisco cisco 4066 Jan  9 02:03 ios-router-rtr1.txt
-rw-rw-r-- 1 cisco cisco 1218 Jan  9 02:03 xr-router-rtr1.txt
```
- View the contents of the generated config files.
```
cisco@ansible-controller:~$ more ios-router-rtr1.txt

cisco@ansible-controller:~$ more xr-router-rtr1.txt
```

### Conclusion

- You can utilize the concept of roles - predetermined order of directories and files to automate generating bulk tasks.

### Example Output

```
cisco@ansible-controller:~$ ansible-playbook config-gen.yml --syntax-check

playbook: config-gen.yml

cisco@ansible-controller:~$ ansible-playbook config-gen.yml

PLAY [Playbook to generate configuration based on role "config-gen"] *********************************************************************************************************************************

TASK [config-gen : Generate the configuration for xr-routers] ****************************************************************************************************************************************
changed: [localhost] => (item={u'timezone_dst': u'EDT', u'gig0000_mask': u'255.255.255.0', u'timezone_offset': -5, u'hostname': u'xr-router-rtr1', u'loopback0_ip': u'192.168.0.1', u'gig0001_mask': u'255.255.255.0', u'timezone': u'EST', u'gig0000_ip': u'10.0.0.5', u'gig0001_ip': u'10.1.0.5', u'loopback0_mask': u'255.255.255.255'})

TASK [config-gen : Generate the configuration for iosxe-routers] *************************************************************************************************************************************
changed: [localhost] => (item={u'timezone_dst': u'EDT', u'areaid': 0, u'ospf_network3': u'10.1.0.0', u'ospf_network1': u'192.168.0.2', u'timezone_offset': -5, u'hostname': u'ios-router-rtr1', u'ospf_network_mask1': u'0.0.0.0', u'ospf_network_mask2': u'0.0.0.255', u'gigaethernet1_mask': u'255.255.255.0', u'gigaethernet1_ip': u'10.0.0.6', u'ospf_network2': u'10.0.0.0', u'timezone': u'EST', u'ospf_network_mask3': u'0.0.0.255', u'loopback0_ip': u'192.168.0.2', u'loopback0_mask': u'255.255.255.255'})

PLAY RECAP *******************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0   

cisco@ansible-controller:~$
```

---

## 3.6 TextFSM Module

---
## 3.6 TextFSM Module
### Objective
- Create a playbook to generate structured command output from standard/generic command output.

### Approach
- This playbook uses an ansible module caled ntc_show which in turn uses a Python module called textfsm
- Create a playbook to collect show command output and let the modules do the parsing and print a structured output


### Lab exercise

#### Step-1: Create an ansible playbook as shown below to print structured output from show vesion brief command

```
cisco@Ansible-Controller:~/project1$ vi p31-runcfg-bkup.yml
---

- name: GET STRUCTURED DATA BACK FROM CLI DEVICES
  hosts: PE1
  connection: local
  gather_facts: False
  
  tasks:

    - name: GET DATA
      ntc_show_command:
        connection: ssh
        platform: cisco_xr
        template_dir: /home/cisco/ntc-ansible/ntc-templates/templates
        use_templates: True 
        command: show version brief
        host: 172.16.122.201
        username: cisco
        password: cisco
      register:  mydata
    -
      name: FULL show version output
      debug: var=mydata
    - 
      name: PRINT Image file location
      debug: msg="Image file location is {{ mydata.response[0].imgloc}}"
    - 
      name: PRINT  IOS-XR SW Version
      debug: msg="IOS-XR SW Version is {{ mydata.response[0].version}}"
    - 
      name: PRINT DEVICE Uptime
      debug: msg="Device Uptime is {{ mydata.response[0].uptime}}"



Brief Explanation about the above playbook:

When using any module other than a raw module, set the connection to local. Gathering off facts is disabled in this playbook.

This playbook uses "ntc_show_command" ansible module which has been pre-loaded in the ansible controller.ntc_show_command module is part of larger suite of Ansible modules and for more information see the below git.

https://github.com/networktocode/ntc-ansible

For some additional information about ntc ansible module (specifically ntc_show_command) options, see the below URL to understand what are the available options, which options are mandatory and how to use/configure playbooks using them.

Some important options that are used in this playbook are explained below to level set the attendees on these options. 

https://ntc-docs.readthedocs.io/en/latest/ntc-ansible%20Modules%20(multi-vendor)/ntc_show_command_module.html

-------------------------------------------------------------------------------------------
|  Parameter  |  required   | default | options     |   Comments                          |
-------------------------------------------------------------------------------------------
| connection  | no           |  ssh   | ssh/offline | connect to device using netmiko     |
|             |              |        |             | or read an offline file for testing |
| platform    |
| template_dir|
|use_templates|
| command     |


A brief overview/explanation of TextFSM Template is covered uring the lecture explaining the structure/format and how to custom create a template.

All needed templates have been copied to the configured template_directory.

Specifically for this exercise, a template by the name "isco_xr_show_version_brief-v3.template" will be used. This template needs to be configured/activated in the index file of the TextFSM template index file. This file resides in the directory where all templates will be copied/installed (i.e ~/ntc-ansible/ntc-templates/templates/).

There are four comma separated values/parts in each and every line of the index file. First component is the template name , second column/component signifies the hostname of device this template will be used (an asterisk indicates, all hosts), third component signifies for which platform/device type this template is ued and the last one shows the actual command this template is applicable for.

Below example, shows that a template with name "cisco_xr_show_version_brief-v3.template" will be used for converting show version brief IOS-XR command against all hosts to a structured output as defined in the template.

cisco@ansible-ctrlr:~$ more ~/ntc-ansible/ntc-templates/templates/index | grep brief-v3
cisco_xr_show_version_brief-v3.template, .*, cisco_xr, sh[[ow]] ver[[sion]] brief
cisco@ansible-ctrlr:~$ 

Looking further in the template "cisco_xr_show_version_brief-v3.template", we can see that three "values" will be gathered from the command show version brief using this template. Refer to the lecture on the structure of the template and how to build it.

cisco@ansible-ctrlr:~$ more ~/ntc-ansible/ntc-templates/templates/cisco_xr_show_version_brief-v3.template 
Value Version ((.*))
Value UPTIME ((.*))
Value IMGloc ((.*))

Start
  ^.+UTC
  ^Cisco IOS XR Software, Version ${Version} 
  ^Copyright\s\(c\).+
  ^ROM: GRUB, Version(.*) 
  ^.*uptime is ${UPTIME} 
  ^System image file is ${IMGloc} -> Record
cisco@ansible-ctrlr:~$ 



#### Step-2: Manually connect to the router and get the "show version brief" CLI command output to identify the critical/essential information to be extracted using the TextFSM template.

cisco@globster-psl:~$ ssh -l cisco 172.16.122.201
password:<>

RP/0/0/CPU0:XRV1#show version brief 
Mon May 13 11:14:21.109 UTC

Cisco IOS XR Software, Version 6.1.2[Default]
Copyright (c) 2016 by Cisco Systems, Inc.

ROM: GRUB, Version 1.99(0), DEV RELEASE

XRV1 uptime is 3 weeks, 5 days, 19 hours, 30 minutes
System image file is "bootflash:disk0/xrvr-os-mbi-6.1.2/mbixrvr-rp.vm"

cisco IOS XRv Series (Pentium Celeron Stepping 3) processor with 8388095K bytes of memory.
Pentium Celeron Stepping 3 processor at 2592MHz, Revision 2.174
IOS XRv Chassis

1 GigabitEthernet
1 Management Ethernet
97070k bytes of non-volatile configuration memory.
866M bytes of hard disk.
2321392k bytes of disk0: (Sector size 512 bytes).
RP/0/0/CPU0:XRV1#

#### Step-3: Run the playbook as shown in the example below.

cisco@ansible-ctrlr:~/ansible$ ansible-playbook pe1-ntc-show-ver.yml 

PLAY [GET STRUCTURED DATA BACK FROM CLI DEVICES] *************************************************************************************************************************

TASK [GET DATA] **********************************************************************************************************************************************************
ok: [PE1]

TASK [FULL show version output] ******************************************************************************************************************************************
ok: [PE1] => {
    "mydata": {
        "changed": false, 
        "failed": false, 
        "response": [
            {
                "imgloc": "\"bootflash:disk0/xrvr-os-mbi-6.1.2/mbixrvr-rp.vm\"", 
                "uptime": "3 weeks, 5 days, 19 hours, 28 minutes", 
                "version": "6.1.2[Default]"
            }
        ], 
        "response_list": []
    }
}

TASK [PRINT Image file location] *****************************************************************************************************************************************
ok: [PE1] => {
    "msg": "Image file location is \"bootflash:disk0/xrvr-os-mbi-6.1.2/mbixrvr-rp.vm\""
}

TASK [PRINT  IOS-XR SW Version] ******************************************************************************************************************************************
ok: [PE1] => {
    "msg": "IOS-XR SW Version is 6.1.2[Default]"
}

TASK [PRINT DEVICE Uptime] ***********************************************************************************************************************************************
ok: [PE1] => {
    "msg": "Device Uptime is 3 weeks, 5 days, 19 hours, 28 minutes"
}

PLAY RECAP ***************************************************************************************************************************************************************
PE1                        : ok=5    changed=0    unreachable=0    failed=0   

cisco@ansible-ctrlr:~/ansible$ 


```
Second playbook to get structured command output for show ipv4 interface brief command:

cisco@ansible-ctrlr:~/ansible$ more pe1-ntc-show-ipv4-int-bri.yml 
---

- name: GET STRUCTURED DATA BACK FROM CLI DEVICES
  hosts: PE1
  connection: local
  gather_facts: False
  
  tasks:

    - name: GET DATA
      ntc_show_command:
        connection: ssh
        #platform: cisco_ios
        platform: cisco_xr
        template_dir: /home/cisco/ntc-ansible/ntc-templates/templates
        #index_file: /home/cisco/ntc-ansible/ntc-templates/templates/index
        #template_dir: /home/cisco/ntc/projects/ntc-ansible/ntc-templates/templates
        use_templates: True 
        command: show ipv4 interface brief 
        host: 172.16.122.201
        username: cisco
        password: cisco
      register:  mydata
    -
      name: FULL show interface brief command output
      debug: var=mydata
    - 
      name: PRINT interface and state of the interface
      debug: msg="Interface name is {{ mydata.response[0].intf }}, Interface state is {{ mydata.response[0].status }}, IPv4 address is {{ mydata.response[0].ipaddr }}, VR
F Name is {{ mydata.response[0].vrfname }}"
    - 
      name: PRINT interface and state of the interface
      debug: msg="Interface name is {{ mydata.response[1].intf }}, Interface state is {{ mydata.response[1].status }}, IPv4 address is {{ mydata.response[1].ipaddr }}, VR
F Name is {{ mydata.response[1].vrfname }}"
    - 
      name: PRINT interface and state of the interface
      debug: msg="Interface name is {{ mydata.response[2].intf }}, Interface state is {{ mydata.response[2].status }}, IPv4 address is {{ mydata.response[2].ipaddr }}, VR
F Name is {{ mydata.response[2].vrfname }}"
    - 
  #    name: PRINT interface and state of the interface
  #    debug: msg="Interface name is {{ mydata.response[3].interface }}, Interface state is {{ mydata.response[3].intf_state }}"
cisco@ansible-ctrlr:~/ansible$ ansible-playbook pe1-ntc-show-ipv4-int-bri.yml 

PLAY [GET STRUCTURED DATA BACK FROM CLI DEVICES] *************************************************************************************************************************

TASK [GET DATA] **********************************************************************************************************************************************************
ok: [PE1]

TASK [FULL show interface brief command output] **************************************************************************************************************************
ok: [PE1] => {
    "mydata": {
        "changed": false, 
        "failed": false, 
        "response": [
            {
                "intf": "Loopback0", 
                "ipaddr": "4.4.4.4", 
                "proto": "Up", 
                "status": "Up", 
                "vrfname": "default"
            }, 
            {
                "intf": "MgmtEth0/0/CPU0/0", 
                "ipaddr": "172.16.122.201", 
                "proto": "Up", 
                "status": "Up", 
                "vrfname": "default"
            }, 
            {
                "intf": "GigabitEthernet0/0/0/0", 
                "ipaddr": "10.10.1.2", 
                "proto": "Up", 
                "status": "Up", 
                "vrfname": "default"
            }
        ], 
        "response_list": []
    }
}

TASK [PRINT interface and state of the interface] ************************************************************************************************************************
ok: [PE1] => {
    "msg": "Interface name is Loopback0, Interface state is Up, IPv4 address is 4.4.4.4, VRF Name is default"
}

TASK [PRINT interface and state of the interface] ************************************************************************************************************************
ok: [PE1] => {
    "msg": "Interface name is MgmtEth0/0/CPU0/0, Interface state is Up, IPv4 address is 172.16.122.201, VRF Name is default"
}

TASK [PRINT interface and state of the interface] ************************************************************************************************************************
ok: [PE1] => {
    "msg": "Interface name is GigabitEthernet0/0/0/0, Interface state is Up, IPv4 address is 10.10.1.2, VRF Name is default"
}

PLAY RECAP ***************************************************************************************************************************************************************
PE1                        : ok=5    changed=0    unreachable=0    failed=0   

cisco@ansible-ctrlr:~/ansible$
























































































































































































___
## 3.7 Yogi Custom Module













































































































































































































---
# 4. Appendix


## 4.1 Ansible Vault
- Ansible Vault is a feature of Ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext.
- Ansible vault has many security features but this section will cover only the basics of encrypting and decrypting a file.

### Objective
- Encrypt the inventory (/etc/hosts) file using Ansible Vault, so router login data is not stored in plain text.

### Lab exercise
- Use Ansible vault to encrypt and decrypt the inventory file.

#### Step-1: Pre-check

- Pay attention the owner and file privileges (`-rw-r--r--`).
- Make sure the playbook runs without any errors

```
$ ls -l /etc/ansible/hosts

$ cat /etc/ansible/hosts

$ ansible-playbook p2-ioscmd.yml
```
#### Step-2: Encrypt inventory file and execute a playbook

- Encrypt the inventory file using `ansible-vault` command.
    - sudo password: `cisco`
    - New vault password: `cisco123`
```
$ ansible-vault --help

$ sudo ansible-vault encrypt /etc/ansible/hosts
```

  - Read the encrypted file

```
$ sudo cat /etc/ansible/hosts

$ sudo ansible-vault view /etc/ansible/hosts
```

- Playbook execution without vault-key will fail.
- Read the failure error message about inventory file

```
$ sudo ansible-playbook p2-ioscmd.yml
```

- Execute the playbook with vault-key
- Input sudo and vault passwords as prompted.

```
$ sudo ansible-playbook p2-ioscmd.yml --ask-vault-pass
```

#### Step-3: Decrypt and restore

- Decrypt the inventory file using `ansible-vault` command.
    - sudo password: `cisco`
    - New vault password: `cisco123`

```
$ sudo ansible-vault decrypt /etc/ansible/hosts
```
- Pay attention the owner and file privileges. Notice that the file permissions are not changed back to the original values.
- Change the file permissions to the previous settings.
- Make sure that the file permissions are: `-rw-r--r--`

```
$ ls -l /etc/ansible/hosts

$ sudo chmod 644 /etc/ansible/hosts

$ ls -l /etc/ansible/hosts
```

- Ensure the inventory file has been decrypted by view its content and executing a playbook.

```
$ cat /etc/ansible/hosts

$ ansible-playbook p2-ioscmd.yml
```
### Conclusion
- Ansible files can be encrypted and decrypted using Vault.
- It is possible to use the encrypted files in playbooks using the option --ask-vault-pass
-Note: When vault encrypted the file it changed the file permissions; but when it decrypted the file it did not restore the old permissions.
- Review the section and discuss if you have any questions.

### Example output

```
cisco@ansible-controller:~$ ls -l /etc/ansible/hosts
-rw-r--r-- 1 root root 1199 Jan  8 00:50 /etc/ansible/hosts
cisco@ansible-controller:~$
cisco@ansible-controller:~$ cat /etc/ansible/hosts
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

[IOS]
R1 ansible_host=172.16.101.191 ansible_user=cisco ansible_ssh_pass=cisco

[XR]
R2 ansible_host=172.16.101.192 ansible_user=cisco ansible_ssh_pass=cisco

[ALL:children]
IOS
XR

cisco@ansible-controller:~$ ansible-playbook p2-ioscmd.yml

PLAY [collect ip route summary from all IOS devices] *************************************************************************************************************************************************

TASK [execute route summary command] *****************************************************************************************************************************************************************
ok: [R1]

TASK [print output] **********************************************************************************************************************************************************************************
ok: [R1] => {
    "P2_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "ospf 1          0           1           0           96          308",
            "  Intra-area: 1 Inter-area: 0 External-1: 0 External-2: 0",
            "  NSSA External-1: 0 NSSA External-2: 0",
            "bgp 1           0           0           0           0           0",
            "  External: 0 Internal: 0 Local: 0",
            "internal        2                                               1048",
            "Total           2           4           0           384         2268"
        ]
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0   


cisco@ansible-controller:~$ ansible-vault --help
Usage: ansible-vault [create|decrypt|edit|encrypt|encrypt_string|rekey|view] [options] [vaultfile.yml]

encryption/decryption utility for Ansible data files

Options:
  --ask-vault-pass      ask for vault password
  -h, --help            show this help message and exit
  --new-vault-id=NEW_VAULT_ID
                        the new vault identity to use for rekey
  --new-vault-password-file=NEW_VAULT_PASSWORD_FILE
                        new vault password file for rekey
  --vault-id=VAULT_IDS  the vault identity to use
  --vault-password-file=VAULT_PASSWORD_FILES
                        vault password file
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
  --version             show program's version number and exit

 See 'ansible-vault <command> --help' for more information on a specific
command.

cisco@ansible-controller:~$ sudo ansible-vault encrypt /etc/ansible/hosts
[sudo] password for cisco:
New Vault password:
Confirm New Vault password:
Encryption successful

cisco@ansible-controller:~$ sudo cat /etc/ansible/hosts
$ANSIBLE_VAULT;1.1;AES256
38376662323536326239666333353931643032646533366635623434653736386461653133363439
3630393135666536623030386561626138646161393030610a376432333762386138393833336634

cisco@ansible-controller:~$ sudo ansible-vault view /etc/ansible/hosts
Vault password:
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

[IOS]
R1 ansible_host=172.16.101.191 ansible_user=cisco ansible_ssh_pass=cisco

[XR]
R2 ansible_host=172.16.101.192 ansible_user=cisco ansible_ssh_pass=cisco

[ALL:children]
IOS
XR

cisco@ansible-controller:~$ sudo ansible-playbook p2-ioscmd.yml
 [WARNING]:  * Failed to parse /etc/ansible/hosts with yaml plugin: Attempting to decrypt but no vault secrets found

 [WARNING]:  * Failed to parse /etc/ansible/hosts with ini plugin: Attempting to decrypt but no vault secrets found

 [WARNING]: Unable to parse /etc/ansible/hosts as an inventory source

 [WARNING]: No inventory was parsed, only implicit localhost is available

 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

 [WARNING]: Could not match supplied host pattern, ignoring: IOS


PLAY [collect ip route summary from all IOS devices] *************************************************************************************************************************************************
skipping: no hosts matched

PLAY RECAP *******************************************************************************************************************************************************************************************

cisco@ansible-controller:~$  sudo ansible-playbook p2-ioscmd.yml --ask-vault-pass
Vault password:

PLAY [collect ip route summary from all IOS devices] *************************************************************************************************************************************************

TASK [execute route summary command] *****************************************************************************************************************************************************************
ok: [R1]

TASK [print output] **********************************************************************************************************************************************************************************
ok: [R1] => {
    "P2_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "ospf 1          0           1           0           96          308",
            "  Intra-area: 1 Inter-area: 0 External-1: 0 External-2: 0",
            "  NSSA External-1: 0 NSSA External-2: 0",
            "bgp 1           0           0           0           0           0",
            "  External: 0 Internal: 0 Local: 0",
            "internal        2                                               1048",
            "Total           2           4           0           384         2268"
        ]
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0   

cisco@ansible-controller:~$ sudo ansible-vault decrypt /etc/ansible/hosts
Vault password:
Decryption successful
cisco@ansible-controller:~$ ls -l /etc/ansible/hosts
-rw------- 1 root root 1199 Jan  9 02:21 /etc/ansible/hosts
cisco@ansible-controller:~$ sudo chmod 644 /etc/ansible/hosts
cisco@ansible-controller:~$ ls -l /etc/ansible/hosts
-rw-r--r-- 1 root root 1199 Jan  9 02:21 /etc/ansible/hosts
cisco@ansible-controller:~$ cat /etc/ansible/hosts
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

[IOS]
R1 ansible_host=172.16.101.191 ansible_user=cisco ansible_ssh_pass=cisco

[XR]
R2 ansible_host=172.16.101.192 ansible_user=cisco ansible_ssh_pass=cisco

[ALL:children]
IOS
XR


cisco@ansible-controller:~$ ansible-playbook p2-ioscmd.yml

PLAY [collect ip route summary from all IOS devices] *************************************************************************************************************************************************

TASK [execute route summary command] *****************************************************************************************************************************************************************
ok: [R1]

TASK [print output] **********************************************************************************************************************************************************************************
ok: [R1] => {
    "P2_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "ospf 1          0           1           0           96          308",
            "  Intra-area: 1 Inter-area: 0 External-1: 0 External-2: 0",
            "  NSSA External-1: 0 NSSA External-2: 0",
            "bgp 1           0           0           0           0           0",
            "  External: 0 Internal: 0 Local: 0",
            "internal        2                                               1048",
            "Total           2           4           0           384         2268"
        ]
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0   

```

---

## 4.2 Optional exercise op23-cmd.yml

### Objective
- Write a playbook, op23-cmd.yml, to meet the below requirements:
  - Collect output of route summary from both IOS and XR routers
  - Use the modules, ios_command and iosxr_command
  - Write 2 plays within one playbook.

### Lab exercise
- Create a playbook called op23-cmd.xml to collect output from IOS and XR router.

```
cisco@ansible-controller:~$ vi op23-cmd.yml
---
  - name: play-1:get route summary from IOS routers
    hosts: IOS
    connection: local

    tasks:
      - name: execute IOS route summary command
        ios_command:
          commands:
            show ip route summary

        register: IOS_OUTPUT

      - name: print output
        debug:
          var: IOS_OUTPUT.stdout_lines

  - name: play-2:get route summary from XR routers
    hosts: XR
    connection: local

    tasks:
      - name: execute XR route summary command
        iosxr_command:
          commands:
            show route summary

        register: XR_OUTPUT

      - name: print output
        debug:
          var: XR_OUTPUT.stdout_lines
```

### Example output
```
cisco@ansible-controller:~$ ansible-playbook op23-cmd.yml --syntax-check

playbook: op23-cmd.yml
cisco@ansible-controller:~$ ansible-playbook op23-cmd.yml

PLAY [play-1:get route summary from IOS routers] *****************************************************************************************************************************************************

TASK [execute IOS route summary command] *************************************************************************************************************************************************************
ok: [R1]

TASK [print output] **********************************************************************************************************************************************************************************
ok: [R1] => {
    "IOS_OUTPUT.stdout_lines": [
        [
            "IP routing table name is default (0x0)",
            "IP routing table maximum-paths is 32",
            "Route Source    Networks    Subnets     Replicates  Overhead    Memory (bytes)",
            "application     0           0           0           0           0",
            "connected       0           3           0           288         912",
            "static          0           0           0           0           0",
            "ospf 1          0           1           0           96          308",
            "  Intra-area: 1 Inter-area: 0 External-1: 0 External-2: 0",
            "  NSSA External-1: 0 NSSA External-2: 0",
            "bgp 1           0           0           0           0           0",
            "  External: 0 Internal: 0 Local: 0",
            "internal        2                                               1048",
            "Total           2           4           0           384         2268"
        ]
    ]
}

PLAY [play-2:get route summary from XR routers] ******************************************************************************************************************************************************

TASK [execute XR route summary command] **************************************************************************************************************************************************************
ok: [R2]

TASK [print output] **********************************************************************************************************************************************************************************
ok: [R2] => {
    "XR_OUTPUT.stdout_lines": [
        [
            "Route Source                     Routes     Backup     Deleted     Memory(bytes)",
            "connected                        1          1          0           320          ",
            "local                            2          0          0           320          ",
            "dagr                             0          0          0           0            ",
            "ospf 1                           1          0          0           160          ",
            "bgp 1                            0          0          0           0            ",
            "Total                            4          1          0           800"
        ]
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=0    unreachable=0    failed=0   
R2                         : ok=2    changed=0    unreachable=0    failed=0   

```
---
## 4.3 Optional exercise op28-conditionals.yml
### Objective
- Create a playbook, which will:
  - Detect router OS
  - If a router has IOS, print message, "\<hostname\> is a IOS router" and if a router has XR, print, "\<hostname\> is a XR router"
  - Playbook needs to find the router names dynamically from the inventory file.

### Lab exercise

- Create a playbook called op28-conditionals.yml to conditional check router os type.

```
cisco@ansible-controller:~$ vi op28-conditionals.yml
---
- name: print each host and its OS
  hosts: ALL

  tasks:
    - name: collect OS
      raw: show ver

      register: OS

    - name: print IOS host
      debug:
        msg: "{{ inventory_hostname }} is an IOS Router."
      when: '"IOS XE" in OS.stdout'

    - name: print XR host
      debug:
        msg: "{{ inventory_hostname }} is an XR Router."
      when: '"IOS XR" in OS.stdout'
```

### Example output

```
cisco@ansible-controller:~$ ansible-playbook op28-conditionals.yml --syntax-check

playbook: op8-conditionals.yml
cisco@ansible-controller:~$ ansible-playbook op28-conditionals.yml

PLAY [print each host and its OS] ********************************************************************************************************************************************************************

TASK [collect OS] ************************************************************************************************************************************************************************************
changed: [R1]
changed: [R2]

TASK [print IOS host] ********************************************************************************************************************************************************************************
skipping: [R2]
ok: [R1] => {
    "msg": "R1 is an IOS Router."
}

TASK [print XR host] *********************************************************************************************************************************************************************************
skipping: [R1]
ok: [R2] => {
    "msg": "R2 is an XR Router."
}

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=2    changed=1    unreachable=0    failed=0   
R2                         : ok=2    changed=1    unreachable=0    failed=0   

```
---
## 4.4 Optional exercise op31-runcfg-bkup.yml (Router Config Backup)
### Objective
- We have two additional requirements on top of the Config Backup playbook that we already did.
  - Write the Running Config to Startup Config on IOS devices
  - Gather the Admin mode Running config from XR devices and save it to a file.

### Lab exercise
- Since these will be run on different subsets of nodes, new plays in the same playbook will be created
  - For writing IOS configs, the "ios_command" module can be used.
  - For gathering XR admin mode config, the “raw” and “copy” modules can be used.
- Create playbook, op31-runcfg-bkup.yml, with the contents below:

```
cisco@ansible-controller:~$ vi op31-runcfg-bkup.yml
---
- name: Get Router Config from All Routers
  hosts: all
  gather_facts: no

  tasks:
    - name: Collect Show run from all routers
      raw: "show run"

      register: RUNCFG

    - set_fact: time="{{lookup('pipe','date \"+%Y-%m-%d-%H-%M\"')}}"

    - name: save output to a file
      connection: local
      copy:
        content="\n ===show run=== \n {{ RUNCFG.stdout }}"
        dest="./{{ inventory_hostname }}_run_cfg_{{ time }}.txt"

- name: Write IOS Running Config to Startup Config
  hosts: IOS
  gather_facts: no
  connection: local

  tasks:
    - name: Write IOS Config
      ios_command:
        authorize: yes
        commands: write memory

- name: Get Admin Config from XR Routers
  hosts: XR
  gather_facts: no

  tasks:
    - name: Collect Admin show run from XR routers
      raw: "admin show run"

      register: ADMINCFG

    - name: save output to a file
      connection: local
      copy:
        content="\n ===admin show run=== \n {{ ADMINCFG.stdout }}"
        dest="./{{ inventory_hostname }}_admin_run_cfg_{{ time }}.txt"
```

### Example output

```
cisco@ansible-controller:~$ ansible-playbook op31-runcfg-bkup.yml --syntax-check

playbook: op31-runcfg-bkup.yml

cisco@ansible-controller:~$ ansible-playbook op31-runcfg-bkup.yml

PLAY [Get Router Config from All Routers] ************************************************************************************************************************************************************

TASK [Collect Show run from all routers] *************************************************************************************************************************************************************
changed: [R1]
changed: [R2]

TASK [set_fact] **************************************************************************************************************************************************************************************
ok: [R1]
ok: [R2]

TASK [save output to a file] *************************************************************************************************************************************************************************
changed: [R2]
changed: [R1]

PLAY [Write IOS Running Config to Startup Config] ****************************************************************************************************************************************************

TASK [Write IOS Config] ******************************************************************************************************************************************************************************
ok: [R1]

PLAY [Get Admin Config from XR Routers] **************************************************************************************************************************************************************

TASK [Collect Admin show run from XR routers] ********************************************************************************************************************************************************
changed: [R2]

TASK [save output to a file] *************************************************************************************************************************************************************************
changed: [R2]

PLAY RECAP *******************************************************************************************************************************************************************************************
R1                         : ok=4    changed=2    unreachable=0    failed=0   
R2                         : ok=5    changed=4    unreachable=0    failed=0   

cisco@ansible-controller:~$ ls -ltr *run_cfg*
-rw-rw-r-- 1 cisco cisco 1359 Jan  8 23:26 R2_run_cfg_2019-01-08-23-26.txt
-rw-rw-r-- 1 cisco cisco 4622 Jan  8 23:26 R1_run_cfg_2019-01-08-23-26.txt
-rw-rw-r-- 1 cisco cisco 1359 Jan  8 23:34 R2_run_cfg_2019-01-08-23-34.txt
-rw-rw-r-- 1 cisco cisco 4622 Jan  8 23:34 R1_run_cfg_2019-01-08-23-34.txt
-rw-rw-r-- 1 cisco cisco 1686 Jan  9 16:59 R2_run_cfg_2019-01-09-16-59.txt
-rw-rw-r-- 1 cisco cisco 4919 Jan  9 16:59 R1_run_cfg_2019-01-09-16-59.txt
-rw-rw-r-- 1 cisco cisco  214 Jan  9 16:59 R2_admin_run_cfg_2019-01-09-16-59.txt
```

---
## 4.5 Optional exercise op33-mop.yml (MOP)

### Objective
- We have  additional requirement on top of the MOP playbook that we already did.
  - Create a playbook that compares the differences between the pre-capture and post-capture files.
  - Create a file with the differences between pre-config and post-config data

### Lab exercise
- Create playbook, op33-diff.yml, to compare the pre-capture and post-capture files.
- For the compare operation, we can use command module to call the Linux diff utility.

```
cisco@ansible-controller:~$ vi op33-diff.yml
---
- name: Compare pre and post files and create a diff file
  hosts: localhost
  connection: local

  tasks:
    - set_fact: TIMESTAMP="{{lookup('pipe','date \"+%Y-%m-%d-%H-%M\"')}}"

    - name: diff pre and post IOS files
      command: diff /home/cisco/R1-OSPF-Precheck.txt /home/cisco/R1-OSPF-Postcheck.txt

      register: IOS_DIFF

      failed_when: "IOS_DIFF.rc > 1"

    - name: create diff file with timestamp included in the name
      copy:
        content: "{{ IOS_DIFF.stdout }}"
        dest: "/home/cisco/op33-ios_diff_{{ TIMESTAMP }}.txt"

    - name: diff pre and post XR files
      command: diff /home/cisco/R2-OSPF-Precheck.txt /home/cisco/R2-OSPF-Postcheck.txt

      register: XR_DIFF

      failed_when: "XR_DIFF.rc > 1"

    - name: create diff file with timestamp included in the name
      copy:
        content: "{{ XR_DIFF.stdout }}"
        dest: "/home/cisco/op33-xr_diff_{{ TIMESTAMP }}.txt"

```

### Example output

```
cisco@ansible-controller:~$ ansible-playbook op33-diff.yml --syntax-check

playbook: op33-diff.yml
cisco@ansible-controller:~$ ansible-playbook op33-diff.yml

PLAY [Compare pre and post files and create a diff file] *********************************************************************************************************************************************

TASK [set_fact] **************************************************************************************************************************************************************************************
ok: [localhost]

TASK [diff pre and post IOS files] *******************************************************************************************************************************************************************
changed: [localhost]

TASK [create diff file with timestamp included in the name] ******************************************************************************************************************************************
changed: [localhost]

TASK [diff pre and post XR files] ********************************************************************************************************************************************************************
changed: [localhost]

TASK [create diff file with timestamp included in the name] ******************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *******************************************************************************************************************************************************************************************
localhost                  : ok=5    changed=4    unreachable=0    failed=0   

cisco@ansible-controller:~$

cisco@ansible-controller:~$ ls -l op33*.txt
-rw-rw-r-- 1 cisco cisco 393 Jan  9 16:08 op33-ios_diff_2019-01-09-16-08.txt
-rw-rw-r-- 1 cisco cisco 489 Jan  9 16:08 op33-xr_diff_2019-01-09-16-08.txt

```

---
## 4.6 Ansible installation
- Ansible control machine is on Linux based systems with Python 2 (versions 2.6 or higher) or Python 3 (versions 3.5 or higher).
- Red Hat, Debian, CentOS, OS X (MAC OS), Ubuntu, BSDs etc. are supported. MS Windows OS is not supported.
- Installation steps are straight forward. Depending on your OS flavor, pick the steps from the installation guide.
- Ansible installation guide: http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
- Just for example, installtion steps for CentOS:

```
$ sudo yum update

$ sudo yum install ansible
```

- Installation steps for Ubuntu:

```
$ sudo apt-get update

$ sudo apt-get install software-properties-common

$ sudo apt-add-repository ppa:ansible/ansible

$ sudo apt-get update

$ sudo apt-get install ansible
```
- Note: Some Ubuntu releases have Ansible packages in their default repositories.  These are usually old and outdated versions, so it is highly recommended to use the above instructions to get the up-to-date version.

---

## 4.7 Reference
- Ansible Documentation: http://docs.ansible.com/ansible/latest/index.html
- YAML Version 1.2 Specs: http://www.yaml.org/spec/1.2/spec.html
- Jinjia2 Templating: http://jinja.pocoo.org/docs/dev/templates/
- Ansible installation: https://www.youtube.com/watch?v=NIEVaCBGOUc
- Introduction to YAML: https://www.youtube.com/watch?v=o9pT9cWzbnI
- Ansible - Playbooks for beginners: https://www.youtube.com/watch?v=Z01b9QZG0D0
- Ansible Quick Reference Guide: https://gist.github.com/andreicristianpetcu/b892338de279af9dac067891579cad7d
- Python Configuration generation: - Kirk Byers Blog – Network configuration using Ansbile
- Ansible Up and Running – Lorin Hochstein

---

**<p align="center">End of Lab Guide</p>**

---
