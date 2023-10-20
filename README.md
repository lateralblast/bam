![alt tag](https://raw.githubusercontent.com/lateralblast/bam/master/images/bam.gif)

BAM
===

BMC Ansible/Automation Module

Version: 0.4.0

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

In addition to the usual Ansible module features, this module has the following features:

- Where a commnd is run it can return the command run making it useful for debugging
  - This can be discovered via the register function and calling the command property
- Can search output for a value which is useful for debugging and doing conditional actions
  - This can be called for in two ways
    - A search function which will return the line the string is found in
    - A searchforvalue function which will try to return just the value in search result rather than the entire line
- Return the value of a get function which is also useful for debugging and doing conditional actions
  - This can be discovered via the register function and calling the value property
- Sylinking to the BMC type to the module file can be used to call the BMC type directly in a stanza

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

You can also use the value property which will try to extract the value from the output, for example:

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
    search:           "Gateway"
    execute:          "{{ execute_get }}"
```

```
- name: Output Gateway
  debug:
    msg:  "{{ output.value }}"

```

This would produce the following output:

```
TASK [Output Gateway] ***************************
ok: [HOSTNAME] => {
    "msg": "192.168.11.254"
}
```

This is more useful with the newer racadm command/object structure, where you would follow the following method:

```
- name: Get Gateway references from iDRAC Network config via ssh using newer method
  bam:
    bmctype:          idrac
    method:           ssh
    bmchostname:      "{{ ansible_host }}"
    bmcusername:      "{{ idrac_username }}"
    usesshkey:        true
    sshkeyfile:       "{{idrac_sshkey_file}}"
    function:         get
    object:           "iDRAC.IPv4.Gateway"
    execute:          "{{ execute_get }}"
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

Legacy racadm support
---------------------

There is support for legacy racadm commands.
The legacy mode can be enabled by setting the function to config or getconfig.

For example, the legacy racadm command to set the boot once device to VCD-DVD:

```
racadm config -g cfgServerInfo -o cfgServerFirstBootDevice VCD-DVD
```

Has the following ansible stanza:

```
- name: Set the first once device to VCD-DVD using legacy racadm format
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     config
    group:        cfgServerInfo
    object:       cfgServerFirstBootDevice
    value:        VCD-DVD
```

The newer command for this is:

```
racadm set iDRAC.serverboot.FirstBootDevice VCD-DV
```

The ansible stanze for using he newer method is:

```
- name: Set the first once device to VCD-DVD using newer racadm format
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     set
    object:       cfgServerFirstBootDevice
    value:        VCD-DVD
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

Set the Lifecyle Controller to collect system inventory on reset using SSH:

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

Get the AMT system Model name from the System information:

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

Execute a command directly:

```
- name: Set one time boot device to BIOS
  bam:
    bmctype:      idrac
    method:       ssh
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     execute
    bmccommand:   racadm set iDRAC.serverboot.FirstBootDevice BIOS
```

Module version of previous command:

```
- name: Set one time boot device to BIOS
  bam:
    bmctype:      idrac
    method:       ssh
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     set
    object:       iDRAC.serverboot.FirstBootDevice
    value:        BIOS
```

Detailed iDRAC Examples
-----------------------

This example steps through the set up of a RAID controller and virtual disks

Reset RAID config:

```
- name: Reset RAID config
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    resetconfig:  RAID.Integrated.1-1
  register: raidreset_status

- name: Create variable for RAID reset status
  debug:
    var: raidreset_status

- name: Create a job for RAID reset
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  jobqueue
    create:       RAID.Integrated.1-1
  register: raidreset_job

- name: Create variable for RAID reset job
  debug:
    var: raidreset_job

- name: Powercycle server to execute RAID controller reset job
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     serveraction
    subfunction:  powercycle
  register: raidreset_reboot
  when: raidreset_job != ''

- name: Create a variable for RAID reset reboot
  debug:
    var: raidreset_reboot

- name: Wait for RAID Reset job to be completed
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     jobqueue
    subfunction:  view
  register: raidreset_jobstatus
  until: raidreset_jobstatus.stdout.find("Running") <= 0 and raidreset_jobstatus.stdout.find("Scheduled") <= 0 and raidreset_jobstatus.stdout.find("New") <= 0
  retries: 20
  delay: 30
  when: '"Server power operation successful" in raidreset_reboot.stdout_lines'
```

Create virtual disks

```
- name: Create a variable for RAID reset job status
  debug: var=raidreset_jobstatus
 
- name: Check whether RAID reset deleted all the Virtual Disks
  bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  get
    object:       vdisks
  register: vdisk_status

- name: Create a variable for vdisk status
  debug:
    var: vdisks_status

- name: Setup Hardware RAID-1 across first two disks
  bam:
    bmctype:          idrac
    method:           racadm
    bmchostname:      192.168.11.238
    bmcusername:      root
    bmcpassword:      calvin
    function:         raid
    createvd:         RAID.Integrated.1-1
    name:             boot
    raidlevel:        r1
    writepolicy:      wt
    readpolicy:       ra
    diskcachepolicy:  default
    stripesize:       64
    pdkey:            Disk.Bay.0:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.1:Enclosure.Internal.0-1:RAID.Integrated.1-1
  register: vdisk1_raid_status
  when:     vdisks_status is search("No virtual disks")

- name: Create a variable for RAID status
  debug:
    var: vdisk1_raid_status

- name: Setup Hardware RAID-10 across next six disks
  bam:
    bmctype:          idrac
    method:           racadm
    bmchostname:      192.168.11.238
    bmcusername:      root
    bmcpassword:      calvin
    function:         raid
    createvd:         RAID.Integrated.1-1
    name:             data 
    raidlevel:        r10
    writepolicy:      wt
    readpolicy:       ra
    diskcachepolicy:  default
    stripesize:       64
    pdkey:            Disk.Bay.2:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.3:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.4:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.5:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.6:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.7:Enclosure.Internal.0-1:RAID.Integrated.1-1
  register: vdisk2_raid_status
  when:     vdisks_status is search("No virtual disks")

- name: Create a variable for RAID status
  debug:
    var: vdisk2_raid_status

- name: Setup Hardware RAID-1 across last two disks
  bam:
    bmctype:          idrac
    method:           racadm
    bmchostname:      192.168.11.238
    bmcusername:      root
    bmcpassword:      calvin
    function:         raid
    createvd:         RAID.Integrated.1-1
    name:             boot
    raidlevel:        r1
    writepolicy:      wt
    readpolicy:       ra
    diskcachepolicy:  backup
    stripesize:       64
    pdkey:            Disk.Bay.8:Enclosure.Internal.0-1:RAID.Integrated.1-1,Disk.Bay.9:Enclosure.Internal.0-1:RAID.Integrated.1-1
  register: vdisk3_raid_status
  when:     vdisks_status is search("No virtual disks")

- name: Create a variable for RAID status
  debug:
    var: vdisk3_raid1_status

- name: Create a job for RAID-1 configuration
  bam:
    bam:
    bmctype:      idrac
    method:       racadm
    bmchostname:  192.168.11.238
    bmcusername:  root
    bmcpassword:  calvin
    function:     raid
    subfunction:  jobqueue
    queue:        RAID.Integrated.1-1
  register:     register: vdisks_raid_job
  failed_when:  disks_raid_job.stdout is seach("ERROR")
  when:         vdisks_status is search("No virtual disks")

- name: Create a variable for RAID configuration job
  debug:
    var: vdisks_raid_job
```

