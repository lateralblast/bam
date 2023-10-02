![alt tag](https://raw.githubusercontent.com/lateralblast/bam/master/images/bam.gif)

BAM
===

BMC Ansible/Automation Module

Version: 0.3.0

Introduction
------------

This is an Ansible module for automating BMCs, e.g. iDRACs.

This was driven out of the need for a reliable and repeatable way to drive BMCs.
As such it uses CLI tools like SSH/racadm rather than web based APIs such as redshift.
I found redshift APIs to be totally unreliable (for example different versions of iDRAC
support different subsets of capability, whereas racadm supported the capability even if
it wasn't exposed by redfish) and thus results were not repeatable.

There are other modules built by others available with similar features e.g.:

https://github.com/1NoOne1/ansible-racadm

This module is based somewhat on the ideas of other modules, 
and on my previous generic virtualisation ansible module:

https://github.com/lateralblast/vamp

This is Ansible module is designed to be generic and extensible.
Other platforms can be relatively easily added if needed, for example ILOMs, ILOs etc.

I'll added support for Intel IME/AMT via web calls like I've done here:

https://github.com/lateralblast/goat

To use the racadm method, racadm is required.

For x86 based Linux, racadm can be installed from the packages provided by Dell:

https://linux.dell.com/repo/community/openmanage/

For non x86 based Linux and MacOS, you can run racadm via docker using the platform flag.

I've written a script to do this:

https://github.com/lateralblast/dracadm/

The goal of this module is to expose an Ansible based DSL that is more easily
translated to and from the racadm command set allowing for more easy implementation
and debugging.

For example the command from the iDRAC manual to convert a disk reset a controller config is:

```
racadm --nocertwarn -r 192.168.11.238 -u root -p calvin storage resetconfig:RAID.Integrated.1-1
```

The Ansible stanza for this would be:

```
- name: Reset RAID Config
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    nocertwarn:   true
    function:     storage
    resetconfig:  RAID.Integrated.1-1
```

Usage
-----

Add path where module is to ---module-path in anisble-playbook command line.

License
-------

This software is licensed as CC-BA (Creative Commons By Attrbution)

http://creativecommons.org/licenses/by/4.0/legalcode


Requirements
------------

The following components are required:

- ansible
- python

Structure
---------

In the case of iDRACs the script tries to support the older and newer implementations of iDRAC and racadm.

For example the older method of enabling IPMI over LAN uses the following method, with a group and an object:

```
racadm config -g cfgIpmiLan -o cfgIpmiLanEnable 1
```

The newer method of enabling IPMI over LAN uses the following method:

```
racadm get iDRAC.IPMILan.Enable Enabled
```

To address this, if the objectgroup tag is used, it will assume the older method.

For example an Ansible stanza that would force/use the first method:

```
- name: Enable IPMI over LAN via SSH using old method
  bam:
    bmctype:      idrac
    method:       ssh
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     set
    objectgroup:  cfgIpmiLan
    object:       cfgIpmiLanEnable
    value:        1
```

An example Ansible stanza that would force/use the newer method:

```
- name: Enable IPMI over LAN via SSH using new method
  bam:
    bmctype:      idrac
    method:       ssh 
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     set
    objectgroup:  iDRAC.IPMILan.Enable
    value:        Enabled
```

Support for SSH also exists allowing the use of keys. 
When using SSH as the method, if no password is given, it will be assumed SSH keys are being used.

Features
--------

Search
------

To help with processing information, I've added a couple of search function tags.

The "search" tag with return any references to a term. 

For example to return any reference to Gateway from the getniccfg command:

```
- name: Get Gateway references from iDRAC Network config via ssh using newer method
  bam:
    bmctype:      idrac
    method:       ssh
    bmchostname:  "{{ ansible_host }}"
    bmcusername:  "{{ idrac_username }}"
    usesshkey:    true
    sshkeyfile:   "{{idrac_sshkey_file}}"
    function:     getniccfg
    search:       "Gateway"
    execute:      "{{ execute_get }}"
```

This will return the line with Gateway in it, e.g.:

```
Gateway              = 192.168.11.254
```

In order to assist with processing, I've added a "searchforvalue" tag with will attempt to clean up the ouput and only return the value.

For example

```
- name: Get Gateway references from iDRAC Network config via ssh using newer method
  bam:
    bmctype:          idrac
    method:           ssh
    bmchostname:      "{{ ansible_host }}"
    bmcusername:      "{{ idrac_username }}"
    usesshkey:        true
    sshkeyfile:       "{{idrac_sshkey_file}}"
    function:         getniccfg
    searchforvalue:   "Gateway"
    execute:          "{{ execute_get }}"
```

This will return:

```
192.168.11.254
```

You can verify this with a debug statement, e.g.:

```
- name: Output Gateway
  debug:
    msg:  "{{ output.stdout }}"

```

This would produce the following output:

```
TASK [Output Gateway] ***************************
ok: [HOSTNAME] => {
    "msg": "192.168.11.254"
}
```

Debugging
---------

I've added some additional modularity/flexible and debugging to the module.

For example, if you want to run in debug mode without running the actual command, set the execute object to no:

```
- name: Get iDRAC object but don't execute function
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     get
    object:       iDRAC.OS-BMC.AdminState
    execute:      no
```

Otherwise, to get the value of a object:


```
- name: Get iDRAC object
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     get
    object:       iDRAC.OS-BMC.AdminState
```

If you want to know what command was actually run to get the information you can use debug, e.g.:

```
- name: Get Virtual Disk Information
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  get
    object:       vdisks
    verbose:      true
  register: pdisks

- name: Output pdisks
  debug:
    msg: "{{ vdisks.command }}"
```

This will produce the following output:

```
TASK [Print Virtual Disk Information] *****************************************************************************************
ok: [192.168.11.238] => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid get vdisks -o"
}
```

An example of iterating through physical disks and convert them to raid capable disks
and output the commands that would be run:

```
- name: Convert physical disks to RAID capable devices
  bam:
    bmctype:        idrac
    method:         racadm
    bmchostname:    192.168.11.238
    bmcusername:    root
    bmcpassword:    calvin
    function:       raid
    converttoraid:  "Disk.Bay.{{ item }}:Enclosure.Internal.0-1:RAID.Integrated.1-1"
    execute:        false
  with_items:
   - 0
   - 1
   - 2
   - 3
   - 4  
   - 5 
   - 6
   - 7  
   - 8
   - 9
  register:           output

- name: Print job command
  debug:
    msg:      "{{ item.command }}"
  with_items: "{{ output.results }}"
  loop_control:
    label: "{{ item.command }}"
```

The output from this example:

```
TASK [Print job command] *************************************************************
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.0:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.0:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.1:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.1:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.2:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.2:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.3:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.3:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.4:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.4:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.5:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.5:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.6:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.6:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.7:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.7:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.8:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.8:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
ok: [192.168.11.238] => (item=racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.9:Enclosure.Internal.0-1:RAID.Integrated.1-1) => {
    "msg": "racadm --nocertwarn -r 192.168.11.238 -u root -p calvin raid converttoraid:Disk.Bay.9:Enclosure.Internal.0-1:RAID.Integrated.1-1"
}
```

Module File Functionality
-------------------------

The engine (virtualisation platform) can be automatically determined from the file name. 

To do this symlink the type to bam:

```
ln -s bam idrac
```

Then do the following:

```
- name: Example with symlinked file to set engine
  idrac:
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     get
    object:       iDRAC.OS-BMC.AdminState
```

The engine and function can be automatically determined from the file name.

To do this symlink the type and function to bam in the following method:

```
ln -s bam bam_get
```

Then do the following:

```
- name: Example with symlinked file to set engine
  bam_get:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    object:       iDRAC.OS-BMC.AdminState
```

Examples
--------

Get physical disk infomation:

```
- name: Get Physical Disk Information
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  get
    object:       pdisks
  register: pdisks

- name: Output Physical Disk Information
  debug:
    msg: "{{ pdisks.stdout_lines }}"
```

Example output:

```
TASK [Ouput Physical Disks] *************************************
ok: [192.168.11.238] => {
    "msg": [
        "Disk.Bay.0:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.1:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.2:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.3:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.4:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.5:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.6:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.7:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.8:Enclosure.Internal.0-1:RAID.Integrated.1-1",
        "Disk.Bay.9:Enclosure.Internal.0-1:RAID.Integrated.1-1"
    ]
}
```

Get virtual disk information:

```
- name: Get Virtual Disk Information
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  get
    object:       vdisks
  register: pdisks

- name: Output pdisks
  debug:
    msg: "{{ vdisks.stdout_lines }}"
```

Example output:

```
TASK [Print Virtual Disk Information] ********
ok: [192.168.11.238] => {
    "msg": [
        "Disk.Virtual.0:RAID.Integrated.1-1",
        "Disk.Virtual.1:RAID.Integrated.1-1",
        "Disk.Virtual.2:RAID.Integrated.1-1"
    ]
}
```

Get verbose virtual disk information:

```
- name: Get Virtual Disk Information
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  get
    object:       vdisks
    details:      true
  register: pdisks

- name: Output pdisks
  debug:
    msg: "{{ vdisks.stdout_lines }}"
```

Example verbose output:

```
TASK [Print Virtual Disk Information] ***********************************************************************************************
ok: [192.168.11.238] => {
    "msg": [
        "Disk.Virtual.0:RAID.Integrated.1-1",
        "   Status                           = Unknown                                  ",
        "   DeviceDescription                = Virtual Disk 0 on Integrated RAID Controller 1",
        "   Name                             = boot                                     ",
        "   RollupStatus                     = Unknown                                  ",
        "   State                            = Online                                   ",
        "   OperationalState                 = Not applicable                           ",
        "   Layout                           = Raid-1                                   ",
        "   Size                             = 893.75 GB                                ",
        "   SpanDepth                        = 1                                        ",
        "   AvailableProtocols               = SATA                                     ",
        "   MediaType                        = SSD                                      ",
        "   ReadPolicy                       = Read Ahead                               ",
        "   WritePolicy                      = Write Back                               ",
        "   StripeSize                       = 64K                                      ",
        "   DiskCachePolicy                  = Default                                  ",
        "   BadBlocksFound                   = NO                                       ",
        "   Secured                          = NO                                       ",
        "   RemainingRedundancy              = 1                                        ",
        "   EnhancedCache                    = Not Applicable                           ",
        "   T10PIStatus                      = Disabled                                 ",
        "   BlockSizeInBytes                 = 512                                      ",
        "Disk.Virtual.1:RAID.Integrated.1-1",
        "   Status                           = Unknown                                  ",
        "   DeviceDescription                = Virtual Disk 1 on Integrated RAID Controller 1",
        "   Name                             = data                                     ",
        "   RollupStatus                     = Unknown                                  ",
        "   State                            = Online                                   ",
        "   OperationalState                 = Not applicable                           ",
        "   Layout                           = Raid-10                                  ",
        "   Size                             = 2681.25 GB                               ",
        "   SpanDepth                        = 1                                        ",
        "   AvailableProtocols               = SATA                                     ",
        "   MediaType                        = SSD                                      ",
        "   ReadPolicy                       = Read Ahead                               ",
        "   WritePolicy                      = Write Back                               ",
        "   StripeSize                       = 64K                                      ",
        "   DiskCachePolicy                  = Default                                  ",
        "   BadBlocksFound                   = NO                                       ",
        "   Secured                          = NO                                       ",
        "   RemainingRedundancy              = 1                                        ",
        "   EnhancedCache                    = Not Applicable                           ",
        "   T10PIStatus                      = Disabled                                 ",
        "   BlockSizeInBytes                 = 512                                      ",
        "Disk.Virtual.2:RAID.Integrated.1-1",
        "   Status                           = Unknown                                  ",
        "   DeviceDescription                = Virtual Disk 2 on Integrated RAID Controller 1",
        "   Name                             = backup                                   ",
        "   RollupStatus                     = Unknown                                  ",
        "   State                            = Online                                   ",
        "   OperationalState                 = Not applicable                           ",
        "   Layout                           = Raid-1                                   ",
        "   Size                             = 3576.38 GB                               ",
        "   SpanDepth                        = 1                                        ",
        "   AvailableProtocols               = SATA                                     ",
        "   MediaType                        = SSD                                      ",
        "   ReadPolicy                       = Read Ahead                               ",
        "   WritePolicy                      = Write Back                               ",
        "   StripeSize                       = 64K                                      ",
        "   DiskCachePolicy                  = Default                                  ",
        "   BadBlocksFound                   = NO                                       ",
        "   Secured                          = NO                                       ",
        "   RemainingRedundancy              = 1                                        ",
        "   EnhancedCache                    = Not Applicable                           ",
        "   T10PIStatus                      = Disabled                                 ",
        "   BlockSizeInBytes                 = 512                                      "
    ]
}
```

Set the Lifecyle Controller to collect system inventory on reset using SSH

```
- name: Set the Lifecyle Controller to collect system inventory on reset
  bam:
    bmctype:      idrac
    method:       ssh
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     get
    objectgroup:  LifecycleController.Embedded.1
    object:       LCAttributes.1#CollectSystemInventoryOnRestart
    value:        Enabled
```

Get the AMT system Model name from the System information

```
- name: Get AMT object value via http(s)
  bam:
    bmctype:      amt
    method:       http
    bmchostname:  "{{ ansible_host }}"
    bmcusername:  "{{ amt_username }}"
    bmcpassword:  "{{ amt_password }}"
    objectgroup:  "system"     
    object:       "model"
    function:     get
    execute:      "{{ execute_get }}"
```

Detailed iDRAC Examples
-----------------------

Insert example(s) here...
