![alt tag](https://raw.githubusercontent.com/lateralblast/bam/master/images/bam.gif)

BAM
===

BMC Ansible/Automation Module

Introduction
------------

This is an Ansible module for automating BMCs, e.g. iDRACs.

This was driven out of the need for a reliable and repeatable way to drive BMCs.
As such it uses CLI tools like SSH/racadm rather than web based APIs such as redshift.
I found redshift APIs to be totally unreliable and thus results were not repeatable.

At the moment this does not work, this is simply a WIP commit, a shell for what will come.
I'm basing this of the work I did with vamp:

https://github.com/lateralblast/vamp

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

I've added some additional modularity/flexible and debugging to the module.

For example, if you want to run in debug mode without running the actual command, set the execute parameter to no:

```
- name: Don't execute function
  bam:
    engine:   racadm
    function: list
    type:     devices
    execute:  no
```

The engine (virtualisation platform) can be automatically determined from the file name. 

To do this symlink the engine to bam:

```
ln -s bam idrac
```

Then do the following:

```
- name: Example with symlinked file to set engine
  racadm:
    function: list
    type:     devices
```

The engine and function can be automatically determined from the file name.

To do this symlink the engine and function to bam in the following method:

```
ln -s bam bam_list
```

Then do the following:

```
- name: Example with symlinked file to set engine
  kvm_list:
    type:    devices 
```

Examples
--------

Insert examples here...

Detailed iDRAC Example
----------------------

Insert example(s) here...
