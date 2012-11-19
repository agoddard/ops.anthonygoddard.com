---
layout: post
title: "Cookbooks as gems."
date: 2011-05-04 17:54
comments: true
categories: 
---

**Update:** Many of the questions below have been resolved in the [0.10.0 release of Chef](http://www.opscode.com/blog/2011/05/02/chef-0-10-0-released/) and Opscode have also provided a [great overview](http://www.opscode.com/blog/2011/05/05/future-of-opscode-cookbooks/) of the current state of the community cookbooks repository. 

**User Story:** 


*   As a sysadmin
    *   I'd like to version cookbooks and download custom cookbooks 
      *   so that I can better manage cookbook dependencies. 

I've been thinking a lot about what the future of the Opscode community cookbooks site / API might look like. Every way I think about it, I keep coming back to a model similar to that of Ruby gems and I'm interested in knowing if this view is shared and to what extent this parallel makes sense. I think the cookbooks site as it stands is great and in some senses, the cookbooks site is really the heart of chef. Without having such an easy path to 'vendor' the apache2 cookbook for example, the experience of first time chef users might not be the wonderful experience it is today. There are however some cases which users might come across which don't (at least obviously) have a solution in the current cookbooks site, and it would be great to see if the community can solve some of these. The scenarios that immediately come to mind are: 
*   "The new Y cookbook broke my X cookbook, I need the X cookbook to be dependent on the *old* version of Y until I can fix it "
*   "My friend just sent me a cookbook she wrote, which depends on a version of the apache2 cookbook which she modified, I'd like to install her apache2 cookbook and use it alongside the default community apache2 cookbook"
*   "I've seen cookbooks on github, such as at 37Signals. Can I also use knife to install and manage these cookbooks?" Rather than get into too much detail about those specific cases, I thought I'd provide an example of what a rubygems-esque cookbook management process might look like 
{% codeblock lang:bash%}
knife cookbook install apache2 # installs the apache2 cookbook along with dependencies 
knife cookbook install apache2 --version=1.02 # installs a legacy version of the apache2 cookbook 
knife cookbook install agoddard-apache2 # installs my fork of the apache2 cookbook along with dependencies 
knife cookbook install agoddard-custom_weird_app # installs my cookbook and dependencies for an obscure app that only I use but which I want to manage the same way 
knife cookbook install custom_weird_app # the above cookbook when the obscure app becomes more mainstream
{% endcodeblock%}
...and how these might look in a recipe...
{% codeblock lang:ruby%}
include_recipe "apache2"
include_recipe "agoddard-apache2"
include_recipe "apache2", "=1.02"
%w{ apache2 agoddard-apache2 apache2-1.02 }.each do |cb|
  depends cb
end 
{% endcodeblock%}
I'm sure there's things I'm missing here, but I'd love to see where this concept leads.. cookbooks already support versioning but I'm not sure if there's a simple way of maintaining two versions and having legacy cookbooks support the older one. It's obviously currently possible to manually install any cookbook you like, though you then loose the added benefits of using the knife site vendor command, such as branching and dependency resolution. Another area this might help is in keeping cookbooks up to date. Currently if a user wants to update a cookbook, they will fork the cookbook, apply their changes and then send the maintainer a pull request. If the maintainer is unavailable, it's hard for the changes to get back to the community. If in this case the user submitting the changes could simply upload their modified cookbook to the cookbooks site, prefixed with their username (to distinguish it from the main / official cookbook) then the cookbook will be available to the community while they wait for the official version to be patched, tested etc. This isn't ideal, but it's more ideal than an update to your chef server breaking an important cookbook which is reliant on a deprecated feature of the server, for example. I'm sure there are many other approaches to this same issue and I'd love to know what others think. The cookbooks site is a fantastic feature and when I look at it, I feel like I'm looking at the beginnings of something big, like github or rubygems and I'm excited to see where it goes.