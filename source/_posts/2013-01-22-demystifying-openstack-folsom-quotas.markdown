---
layout: post
title: "Demystifying OpenStack Folsom quotas"
date: 2013-01-22 00:04
comments: true
categories: OpenStack
---

With the changes introduced in OpenStack Folsom, there are a few areas where quotas can trip you up if you're not careful, both in Nova and in Cinder.

#Nova Quotas

When setting quotas for users or tenants in Folsom, you need to specify the UUID of the user/tenant.

    agoddard@control1:~# keystone tenant-list
    +----------------------------------+---------+---------+
    |                id                |   name  | enabled |
    +----------------------------------+---------+---------+
    | 040113b0d7824477aacf4ce39df69344 | service |   True  |
    | 9af32c41cee140779d50afb2bc93e322 |   ops   |   True  |
    +----------------------------------+---------+---------+

    agoddard@control1:~# nova quota-show 9af32c41cee140779d50afb2bc93e322
    +-----------------------------+--------+
    | Property                    | Value  |
    +-----------------------------+--------+
    | cores                       | 500    |
    | floating_ips                | 100    |
    | gigabytes                   | 50000  |
    | injected_file_content_bytes | 10240  |
    | injected_files              | 5      |
    | instances                   | 500    |
    | metadata_items              | 128    |
    | ram                         | 200000 |
    | volumes                     | 5000   |
    +-----------------------------+--------+

The tricky part is that setting or viewing the quota using a name will appear to work but it won't actually reference the tenant (or any tenant for that matter):

    agoddard@control1:~# nova quota-show ops
    +-----------------------------+-------+
    | Property                    | Value |
    +-----------------------------+-------+
    | cores                       | 500   |
    | floating_ips                | 10    |
    | gigabytes                   | 50000 |
    | injected_file_content_bytes | 10240 |
    | injected_files              | 5     |
    | instances                   | 500   |
    | metadata_items              | 128   |
    | ram                         | 51200 |
    | volumes                     | 10    |
    +-----------------------------+-------+

As you can see, floating IPs, ram and volumes are all different when using the name rather than UUID.

It get's a little stranger - what if we just make up a tenant:

    agoddard@control1:~# nova quota-show nonexistent-tenant
    +-----------------------------+-------+
    | Property                    | Value |
    +-----------------------------+-------+
    | cores                       | 20    |
    | floating_ips                | 10    |
    | gigabytes                   | 1000  |
    | injected_file_content_bytes | 10240 |
    | injected_files              | 5     |
    | instances                   | 10    |
    | metadata_items              | 128   |
    | ram                         | 51200 |
    | volumes                     | 10    |
    +-----------------------------+-------+
    agoddard@control1:~# nova quota-update nonexistent-tenant --cores 100
    agoddard@control1:~# nova quota-show nonexistent-tenant
    +-----------------------------+-------+
    | Property                    | Value |
    +-----------------------------+-------+
    | cores                       | 100   |
    | floating_ips                | 10    |
    | gigabytes                   | 1000  |
    | injected_file_content_bytes | 10240 |
    | injected_files              | 5     |
    | instances                   | 10    |
    | metadata_items              | 128   |
    | ram                         | 51200 |
    | volumes                     | 10    |
    +-----------------------------+-------+
Nova created an entry for the tenant, even though the tenant doesn't exist. This is what makes it appear that a change has been made when it hasn't, and without an error being thrown, things can get pretty confusing.

**Long story short, always use the UUID of a tenant when setting quotas.**





#Cinder Quotas
I recently found myself having to increase cinder volume quotas for a tenant, from 50TB to 60TB, and got caught in a similar situation to above, however in this case even the UUID's wouldn't work for me.

First, we can check the tenant's quota, which is 50TB:


    agoddard@control1:~# nova quota-show 9af32c41cee140779d50afb2bc93e322
    +-----------------------------+--------+
    | Property                    | Value  |
    +-----------------------------+--------+
    | cores                       | 500    |
    | floating_ips                | 100    |
    | gigabytes                   | 50000  |
    | injected_file_content_bytes | 10240  |
    | injected_files              | 5      |
    | instances                   | 500    |
    | metadata_items              | 128    |
    | ram                         | 200000 |
    | volumes                     | 5000   |
    +-----------------------------+--------+

Now, we simply increase the quota, remembering to use the UUID so the change takes effect.
    
    agoddard@control1:~# nova quota-update 9af32c41cee140779d50afb2bc93e322 --gigabytes 60000

And we can confirm the change:

    agoddard@control1:~# nova quota-show 9af32c41cee140779d50afb2bc93e322
    +-----------------------------+--------+
    | Property                    | Value  |
    +-----------------------------+--------+
    | cores                       | 500    |
    | floating_ips                | 100    |
    | gigabytes                   | 60000  |
    | injected_file_content_bytes | 10240  |
    | injected_files              | 5      |
    | instances                   | 500    |
    | metadata_items              | 128    |
    | ram                         | 200000 |
    | volumes                     | 5000   |
    +-----------------------------+--------+

It appears like everything is fine, however when I try to add additional volume space about 50TB, I get an API error about exceeding my quota, and Horizon shows the quota as 50TB.
By checking the quota with the `cinder` command, we can see that the quota update didn't actually have any effect at all:

    agoddard@control1:~# cinder quota-show 9af32c41cee140779d50afb2bc93e322
    +-----------+-------+
    |  Property | Value |
    +-----------+-------+
    | gigabytes | 50000 |
    |  volumes  |   10  |
    +-----------+-------+

By running `cinder quota-update` rather than `nova quota-update`, the correct quota is set:

    agoddard@control1:~# cinder quota-update 9af32c41cee140779d50afb2bc93e322 --gigabytes=60000
    agoddard@control1:~# cinder quota-show 9af32c41cee140779d50afb2bc93e322
    +-----------+-------+
    |  Property | Value |
    +-----------+-------+
    | gigabytes | 60000 |
    |  volumes  |   10  |
    +-----------+-------+

**Long story short, when setting Cinder quotas, always use `cinder quota-update`, and not `nova quota-update`**.

Both of the above issues seem to be artifacts from changes that Folsom brought about, such as separating volumes from nova. A bugfix for the wrong quota being shown in horizon has been [accepted](https://bugs.launchpad.net/horizon/+bug/1095876) for the [Grizzly-3 milestone](https://launchpad.net/horizon/+milestone/grizzly-3), due on Feb 21.