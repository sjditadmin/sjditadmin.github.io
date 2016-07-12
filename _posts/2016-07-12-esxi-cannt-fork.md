---
layout: post
title:  "esxi error---can't fork"
date:   2016-07-11 11:06:05
categories: vmware
excerpt: esxi error---can't fork 
---

ESXi host cannot initiate vMotion or enable services and reports the error: Heap globalCartel-1 already at its maximum size.Cannot expand (2085618)

Symptoms
Cannot perform a vMotion to and from an ESXi host
Cannot enable services from or to the ESXi host
When attempting to enable services or vMotion the ESXi host fails
When powering on a virtual machine, you see the error:

VMK_NO_MEMORY

When logging in to the ESXi shell, you see the message:

can't fork

When pressing Alt+F12 at the DCUI, you see the error:

WARNING: Heap: 2677: Heap globalCartel-1 already at its maximum size. Cannot expand.

In the /var/log/vmkwarning log file, you see entries similar to:

YYYY-MM-DD TIME cpu18:4287108)WARNING: Heap: 3058: Heap_Align(globalCartel-1, 136/136 bytes, 8 align) failed.  caller: 0x41802a2ca2fd
YYYY-MM-DD TIME cpu8:4287111)WARNING: Heap: 2677: Heap globalCartel-1 already at its maximum size. Cannot expand.
YYYY-MM-DD TIME cpu8:4287111)WARNING: Heap: 3058: Heap_Align(globalCartel-1, 136/136 bytes, 8 align) failed.  caller: 0x41802a2ca2fd
YYYY-MM-DD TIME cpu1:4287132)WARNING: Heap: 2677: Heap globalCartel-1 already at its maximum size. Cannot expand.
YYYY-MM-DD TIME cpu1:4287132)WARNING: Heap: 3058: Heap_Align(globalCartel-1, 136/136 bytes, 8 align) failed.  caller: 0x41802a2ca2fd
YYYY-MM-DD TIME cpu24:4287142)WARNING: Heap: 2677: Heap globalCartel-1 already at its maximum size. Cannot expand.
YYYY-MM-DD TIME cpu24:4287142)WARNING: Heap: 3058: Heap_Align(globalCartel-1, 136/136 bytes, 8 align) failed.  caller: 0x41802a2ca2fd
YYYY-MM-DD TIME cpu5:4025029)WARNING: Heap: 2677: Heap globalCartel-1 already at its maximum size. Cannot expand.
Cause
This issue occurs with ESXi hosts running on HP hardware with these versions of AMS:
hp-ams 500.9.6.0-12.434156
hp-ams-550.9.6.0-12.1198610
hp-ams 500.10.0.0-18.434156
hp-ams-550.10.0.0-18.1198610
Resolution
This is a known issue affecting ESXi 5.x.

To resolve the issue, upgrade to AMS version 10.0.1.

For ESXi 5.0 and 5.1, see HP Agentless Management Service Offline Bundle for VMware ESXi 5.0 and vSphere 5.1.

For ESXi 5.5, see:
HP Agentless Management Service Offline Bundle for VMware vSphere 5.5
HP ESXi Offline Bundle for VMware vSphere 5.5
 
Note: The preceding links were correct as of 2 March 2015. If you find a link is broken, provide a feedback and a VMware employee will update the link.
 
To work around this issue, remove the package on all hosts running on one of the preceding AMS versions.

Note: In some cases, commands running on the ESXi host fails with cant't fork.  In this case the virtual machines running on the ESXi host needs to be powered off and the ESXi host rebooted.

To verify the installed versions of AMS, run this command:

esxcli software vib list | grep ams
 
To remove the package on all hosts running on these AMS versions:
Log in to the host using SSH. For more information, see Using ESXi Shell in ESXi 5.x (2004746).
Run this command to stop the HP service (does not persist on reboot): 

/etc/init.d/hp-ams.sh stop
 
Run this command to remove the VIB: 

esxcli software vib remove -n hp-ams 

Reboot the host.