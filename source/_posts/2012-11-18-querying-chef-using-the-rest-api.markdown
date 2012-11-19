---
layout: post
title: "Querying Chef using the REST API"
date: 2010-12-09 17:54
comments: true
categories: 
author: Chuck Ha
author: Anthony Goddard
---


Assumptions: You have a working knife configuration.

The initial user story that prompted me to figure out how to interact with Chef Server REST API was this:


*   As a System Administrator
  *   I want to be able to see what IP addresses are being used 
  *   In order for me to correctly assign a free IP to a new machine.


In order to access the Chef Server programmatically I first had to figure out authentication. Unfortunately, I couldn't find any good examples of how to actually do this. The Chef Server API wiki page has some basic requirements and concepts, but no concrete examples.

However, it gave me a good starting place--the Chef::REST library. The REST library that comes bundled with Chef in every version > 0.9.0. Let's just make sure that we actually have that version:

{% codeblock lang:bash%}
$ knife --version
Chef: 0.9.12
{% endcodeblock%}

Now that we know the version is ok we’re basically done. All we have to do is load the knife configuration file and then we can use the built in rest library. Here’s the code I ended up with:

{% codeblock lang:ruby%}
require 'bundler/setup'
require 'chef'

Chef::Config.from_file("/path/to/knife.rb")
rest = Chef::REST.new(“http://host:port”)

nodes = rest.get_rest(“/nodes”)
{% endcodeblock%}

Now you can do something like:

{% codeblock lang:ruby%}
nodes.keys.each do |key|
  puts rest.get_rest(“/nodes/#{key}”)[:name]
end
{% endcodeblock%}

So now we just print each IP address that is being used and we can figure out the next free IP address in any given range. That completes this user story!

There was only one gotcha in the whole process. Trying to print a node just throws an ArgumentError:
{% codeblock lang:ruby%}
puts node #=> ArgumentError: Attribute to_ary is not defined!
{% endcodeblock%}