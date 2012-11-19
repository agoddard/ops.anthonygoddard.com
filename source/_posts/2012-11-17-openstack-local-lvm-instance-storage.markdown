---
layout: post
title: "Openstack local LVM instance storage"
date: 2012-11-17 22:28
comments: true
categories: openstack
---
I've been playing with OpenStack on and off since it was released, but recently I had the opportunity to finally build a production cluster. One of our requirements was to keep our storage as fast as possible, and we already had a bunch of hosts with quick disks, so this meant keeping instance storage on local disk and using raw disk backed VMs rather than file backed VMs. While it's always been easy to attach local disk to VMs, doing it automatically through orchestration tools hasn't been simple.  As of the latest release (Folsom), OpenStack supports the provisioning of instance storage onto local LVM volumes, which is exactly what we needed. In order to configure local LVM storage for instances. I've read a few different docs that describe how to do it, but they seem to use different syntax, the following is what worked for me:


{% codeblock nova.conf on compute node lang:bash%}
libvirt_images_type=lvm
libvirt_images_volume_group=nova_local
{% endcodeblock %}

Any compute node you want to use local storage on requires those lines in the nova.conf file. You will also need to create a local LVM volume group called "nova_local" which can be done as follows.

{% codeblock lang:bash%}
# make sure /dev/sda1 is a free disk, formatted as "Linux LVM"
pvcreate /dev/sda1 #create an LVM physical volume from the disk
vgcreate nova_local /dev/sda1 #create the volume group
{% endcodeblock%}

Running `vgs` should now show a "nova_local" volume group.


Nova will by default store disk image files in the /var/lib/nova/images directory, so these LVM volumes aren't any more susceptible to local disk failures as the default configuration, but many people will mount shared storage to the nova images directory for high availability. In my case, I opted for high availability of persistent storage (through OpenStack Cinder), and performance on local stoage. One of the main reasons for this is that the bulk of our infrastructure can be quickly rebuilt by Chef, so in this case day to day performance trumps high availability.

__Update:__ If you're running OpenStack on Ubuntu 12.04+, or CentOS 6.2, there's a [bug](https://answers.launchpad.net/nova/+question/211112) which prevents LVM volumes being deleted when you attempt delete an instance. Until a patch is released for OpenStack, the workaround is to patch `/usr/lib/python2.7/dist-packages/nova/virt/libvirt/utils.py` as follows:

{% codeblock /usr/lib/python2.7/dist-packages/nova/virt/libvirt/utils.py https://answers.launchpad.net/nova/+question/211112 lang:diff %}
- out, err = execute('lvs', '--noheadings', '-o', 'lv_path', vg,
+ out, err = execute('lvs', '--noheadings', '-o', 'lv_name', vg,
{% endcodeblock %}
