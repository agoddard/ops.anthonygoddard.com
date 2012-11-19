---
layout: post
title: "Updating rubygems for chef-client on debian lenny"
date: 2011-01-21 17:54
comments: true
categories: Chef
---

User Story:


As a sysadmin
I need the latest rubygems package
so that chef-client and other tools that rely on rubygems will work.

We have a host which we needed to run chef-client on, unfortunately, the version of rubygems installed by apt was 1.2.0, which resulted in the following error:
{% codeblock lang:bash%}
# chef-client
[Thu, 20 Jan 2011 20:45:43 +0000] INFO: Starting Chef Run (Version 0.9.12)
[Thu, 20 Jan 2011 20:45:43 +0000] WARN: Missing gem 'mysql'
[Thu, 20 Jan 2011 20:45:44 +0000] ERROR: Running exception handlers
[Thu, 20 Jan 2011 20:45:44 +0000] ERROR: Exception handlers complete
/usr/lib/ruby/1.8/chef/provider/package/rubygems.rb:89:in `with_gem_sources': undefined method `sources=' for Gem:Module (NoMethodError)
from /usr/lib/ruby/1.8/chef/provider/package/rubygems.rb:206:in `candidate_version_from_remote'
from /usr/lib/ruby/1.8/chef/provider/package/rubygems.rb:373:in `candidate_version'
from /usr/lib/ruby/1.8/chef/provider/package.rb:44:in `action_install'
from /usr/lib/ruby/1.8/chef/resource.rb:395:in `send'
from /usr/lib/ruby/1.8/chef/resource.rb:395:in `run_action'
from /var/cache/chef/cookbooks/mysql/recipes/client.rb:62:in `from_file'
from /usr/lib/ruby/1.8/chef/cookbook_version.rb:472:in `load_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:40:in `include_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `each'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `include_recipe'
from /var/cache/chef/cookbooks/mysql/recipes/server.rb:20:in `from_file'
from /usr/lib/ruby/1.8/chef/cookbook_version.rb:472:in `load_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:40:in `include_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `each'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `include_recipe'
from /var/cache/chef/cookbooks/drupal/recipes/default.rb:23:in `from_file'
from /usr/lib/ruby/1.8/chef/cookbook_version.rb:472:in `load_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:40:in `include_recipe'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `each'
from /usr/lib/ruby/1.8/chef/mixin/language_include_recipe.rb:27:in `include_recipe'
from /usr/lib/ruby/1.8/chef/run_context.rb:94:in `load'
from /usr/lib/ruby/1.8/chef/run_context.rb:91:in `each'
from /usr/lib/ruby/1.8/chef/run_context.rb:91:in `load'
from /usr/lib/ruby/1.8/chef/run_context.rb:55:in `initialize'
from /usr/lib/ruby/1.8/chef/client.rb:166:in `new'
from /usr/lib/ruby/1.8/chef/client.rb:166:in `run'
from /usr/lib/ruby/1.8/chef/application/client.rb:222:in `run_application'
from /usr/lib/ruby/1.8/chef/application/client.rb:212:in `loop'
from /usr/lib/ruby/1.8/chef/application/client.rb:212:in `run_application'
from /usr/lib/ruby/1.8/chef/application.rb:62:in `run'
from /usr/bin/chef-client:26
{% endcodeblock%}
A quick post to the opscode discussion forum (actually the wrong forum but jtimberman was kind enough to help out) told us that the error was the result of the outdated rubygems package. The easy solution was to simply use rubygem's own self-updating mechanism by running `gem update --system`. Easy enough, however the version installed by apt doesn't actually permit this:
{% codeblock lang:bash%}
ERROR:  While executing gem ... (RuntimeError)
    gem update --system is disabled on Debian. RubyGems can be updated using the official Debian repositories by aptitude or apt-get.
{% endcodeblock%}
At this point we could have introduced new apt sources to see if they contained more recent versions, but we decided we really wanted rubygems to be able to update itself. Rather than reinstall rubygems from source, we found that there was a rubygem on rubygems.org for exactly this purpose "rubygems-update". Unfortunately for some reason, perhaps an incompatibility between rubygems 1.2.0 and rubygems.org, even after adding rubygems.org as a gem source, gem complained it was unable to find the repository online. Downloading the gem and installing it locally provided the fix for that:
{% codeblock lang:bash%}
gem -v
#1.2.0
wget http://production.cf.rubygems.org/gems/rubygems-update-1.4.2.gem
gem install rubygems-update-1.4.2.gem --local
cd /var/lib/gems/1.8/bin
./update_rubygems 
gem -v
# 1.4.2
{% endcodeblock%}
Problem solved. With the new version of rubygems installed, we're now able to run gem update --system to stay up to date.

I believe rubygems was originally installed as a dependency of the chef package in the opscode apt repo, and the rubygems version in there is 1.2.0. So it appears that this problem would always occur when bootstrapping a new debian lenny machine with chef-client using only the apt packages, so I'll be switching to bootstrapping via the chef gems rather than using apt. I'll pose this question to the smart folks at opscode though and see if there isn't a simple answer to the package dependency issue.