![alt tag](https://raw.githubusercontent.com/lateralblast/bam/master/images/bam.gif)

BAM
===

BMC Ansible/Automation Module

Version: 0.1.3

Introduction
------------

This is an Ansible module for automating BMCs, e.g. iDRACs.

This was driven out of the need for a reliable and repeatable way to drive BMCs.
As such it uses CLI tools like SSH/racadm rather than web based APIs such as redshift.
I found redshift APIs to be totally unreliable and thus results were not repeatable.

https://github.com/lateralblast/vamp

This is Ansible module is designed to be generic and extensible.
Other platforms can be relatively easily added if needed, for example ILOMs, ILOs etc.

I'll probably add support for Intel IME/AMT via web calls like I've done here:

https://github.com/lateralblast/goat

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
======

To help with processing infomration, I've added a couple of search function tags.

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

This will return:

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

Debugging
=========

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


Set the Lifecyle Controller to collect system inventory on reset using SSH

```
- bam:
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

Detailed iDRAC Example
----------------------

Insert example(s) here...
