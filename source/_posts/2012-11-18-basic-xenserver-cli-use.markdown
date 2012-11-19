---
layout: post
title: "Basic XenServer CLI use"
date: 2010-12-10 17:54
comments: true
categories: XenServer
---

I recently found myself in a situation where I had to manage a XenServer resource pool over SSH. XenServer's 'xsconsole' tool provided me with a lot of options, but none that would allow me to boot a guest which was powered off.

User Story:

As a sysadmin
I need to start a XenServer guest from the command line
so that I can start guests without having to use a GUI.

When using Resource Pools, there is no way within the console to boot machines which are powered off - attempting to view all machines results in the error message: "This feature is unavailable in Pools with more than 100 Virtual Machines", even if you have less than 100 virtual machines.

Here's where the XenServer command line applications come in handy. In order to boot a machine which is powered off, you can ssh to any machine in the pool and run `xe vm-list`

{% codeblock lang:bash%}
[root@ubio-vmh07 ~]# xe vm-list
uuid ( RO): 0ebb9d7d-1743-f9a0-f5b8-692930cc3ad0
name-label ( RW): app10
power-state ( RO): halted
[root@ubio-vmh07 ~]#
{% endcodeblock %}

This will show you all of your machines by name, state and uuid. Once you find the machine you want to boot, simply run `xe vm-start` and pass it the uuid you want to boot:
[code lang='bash']
[root@ubio-vmh07 ~]# xe vm-start uuid=0ebb9d7d-1743-f9a0-f5b8-692930cc3ad0
[/code]

and the machine will boot right up. You can then continue to manage the VM using xe or via the xsconsole on the host which is running the VM.

The xe help will show you a list of available commands:

{% codeblock lang:bash%}
[root@ubio-vmh07 ~]# xe help --all
{% endcodeblock%}

