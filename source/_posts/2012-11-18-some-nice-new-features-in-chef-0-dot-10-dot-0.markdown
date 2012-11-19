---
layout: post
title: "Some nice new features in Chef 0.10.0"
date: 2011-05-10 17:54
comments: true
categories: 
---

No sooner than I drafted my [last post](http://crankstations.com/cookbooks-as-gems), Opscode released released a huge update to Chef, 0.10.0. Being on on vacation during the lead up to the release, I missed a few of the pre-release announcements, so while I'd heard that some big features, such as [environments](http://www.opscode.com/blog/2011/04/21/chef-0-10-preview-environments/) were coming, I hadn't realized how many cool features were due for release in 0.10.0 and I thought I'd mention a few of my favorite and perhaps lesser known new features here. 

## Knife Plugins 

Co-incidentally, the 0.10.0 release came out the same week that we started work on our first knife plugin, and the new architecture has made the process very straightforward. You don't need to delve into to depths of your gems to extend knife, you can simply throw the plugin in a .chef/plugins/knife directory in your home folder or in your cookbook repo. There's a quick guide to get you started on the [Opscode blog](http://www.opscode.com/blog/2011/04/22/chef-0-10-preview-knife-plugins-and-ui/). 

## Encrypted Data Bags

One of the first questions I get asked when talking about Chef generally relates to security. The new release being able to encrypt the contents of data bags is a big step forward on this front. This means you can now encrypt sensitive values which are stored in your data bags, such as database passwords. To keep things secure, you can also configure the decryption keys at a node level so that only nodes that should have access to the data can see it. [@lusis](http://twitter.com/lusis) also added a [patch](https://github.com/opscode/chef/pull/77) to support for storing decryption keys at a URL instead of a local file, a nice addition which should find its way into an upcoming release. 

## Chef Expander

This behind the scenes update will probably stay out of your way for the most part, but it's cool all the same. Indexing is now taken care of by a new tool called [chef-expander](http://wiki.opscode.com/display/chef/Chef+Indexer#ChefIndexer-ChefExpander), which replaces the old chef-solr-indexer. The cool thing about chef-expander is the ability to setup a cluster of worker nodes to farm out the indexing process. Small installations are fine with just one process running, but it's nice to know that this can scale horizontally as your infrastructure grows. 

## Cookbook Versioning

In my recent post [cookbooks as gems](http://crankstations.com/cookbooks-as-gems) I mentioned a few features I'd like to see in the cookbooks architecture, the main one being a straightforward way of managing different versions of cookbooks. Version 0.10 addresses this and more, including the ability to freeze cookbooks when uploading to the Chef server and the option to set cookbook version constraints in environments. Judging by the upgrades to the cookbook features in knife and their latest [post on the community cookbook site](http://www.opscode.com/blog/2011/05/05/future-of-opscode-cookbooks/) cookbooks management is definitely a high priority in Chef I'm very excited to see how this evolves as Chef moves beyond 0.10 

## Upgrading

If the above features aren't a good enough argument to upgrade straight away, check out the [release notes](http://www.opscode.com/blog/2011/05/02/chef-0-10-0-released/) for a full account of what 0.10.0 brings to the table. The 0.10.0 server is compatible with 0.9.x clients, and the process for upgrading both the server and clients is trivial. Instructions can be found on the [Opscode wiki](http://wiki.opscode.com/display/chef/Upgrading+Chef+0.9.x+to+Chef+0.10.x).