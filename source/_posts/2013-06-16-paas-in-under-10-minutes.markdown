---
layout: post
title: "PaaS in under 10 minutes"
date: 2013-06-16 22:58
comments: true
categories: 
---

PaaS in under 10 minutes (or, "Let's see if building a PaaS and deploying an app is faster than formatting a disk").

We recently had some 4TB (RAID-0) external drives shipped to us to fill with BHL(biodiversitylibrary.org) content, we needed to format them as ext4 and then fill them up with books. Formatting the disks took a while, so I decided that for the next disk, I would see if I could setup a PaaS using Dokku, a cool little set of bash scripts that do some proxy, building & assorted magic on top of docker. And then deploy an app to it. Here's how it panned out.

#start the ext4 format:
    [16:10:41] [root@clustr-03 /root]# date;mkfs.ext4 /dev/sdz1
    Thu Jun 13 16:12:35 EDT 2013

    Writing inode tables:  1/29809


#prep the server with dokku
    $ wget -qO- https://raw.github.com/progrium/dokku/master/bootstrap.sh | sudo bash

We'll let that bake for a little while

#setup app
    ~/dev $ date;mkdir awyeah
    Thu Jun 13 16:12:55 EDT 2013
    ~/dev $ cd awyeah/
    ~/dev/awyeah $ bundle init
    ~/dev/awyeah $ touch web.rb
    ~/dev/awyeah $ touch config.ru
    ~/dev/awyeah $ vim Gemfile
{% codeblock web.rb lang:ruby%}
    source "https://rubygems.org"
    ruby "1.9.3"
    gem 'sinatra'
{% endcodeblock %}
    ~/dev/awyeah $ vim web.rb
{% codeblock web.rb lang:ruby%}

require 'sinatra'

get '/' do
  "awwwwyeah!!!"
end
{% endcodeblock %}

#time check
    ~/dev/awyeah $ date;vim config.ru
    Thu Jun 13 16:14:05 EDT 2013
    
{% codeblock config.ru lang:ruby%}
require './web.rb'
run Sinatra::Application
{% endcodeblock %}

    ~/dev/awyeah $ git init .
    Initialized empty Git repository in /Users/agoddard/dev/awyeah/.git/
    ~/dev/awyeah (master) $ git add .
    ~/dev/awyeah (master) $ git ci -am "awwyeah initial commit"
    [master (root-commit) ade6ea2] awwyeah initial commit
     3 files changed, 10 insertions(+)
     create mode 100644 Gemfile
     create mode 100644 config.ru
     create mode 100644 web.rb
    ~/dev/awyeah (master) $ git remote add deploy git@deploy.eol.org:awwyeah
    ~/dev/awyeah (master) $ bundle
    Fetching gem metadata from https://rubygems.org/...........
    Fetching gem metadata from https://rubygems.org/..
    Resolving dependencies...
    Installing rack (1.5.2)
    Installing rack-protection (1.5.0)
    Installing tilt (1.4.1)
    Installing sinatra (1.4.3)
    Using bundler (1.3.5)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.
    ~/dev/awyeah (master) $ git ci -am "added Gemfile.lock"
    [master 787cdaa] added Gemfile.lock
     1 file changed, 17 insertions(+)
     create mode 100644 Gemfile.lock
#back to server
    [...]
    nginx-reloader start/running, process 26946
    Be sure to upload a public key for your user:
     cat ~/.ssh/id_rsa.pub | ssh root@deploy.eol.org "gitreceive upload-key ag"

looks good, let's finish it off

    cat ~/.ssh/id_rsa.pub | ssh root@deploy.eol.org "gitreceive upload-key ag"

#ship it!
    ~/dev/awyeah (master) $ git push deploy master
    Counting objects: 4, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 500 bytes | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    remote: -----> Building awwyeah ...
    remote:        Ruby/Rack app detected
    remote: -----> Using Ruby version: ruby-1.9.3
    remote: -----> Installing dependencies using Bundler version 1.3.2
    remote:        Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin --deployment
    remote:        Fetching gem metadata from https://rubygems.org/..........
    remote:        Fetching gem metadata from https://rubygems.org/..
    remote:        Installing rack (1.5.2)
    remote:        Installing rack-protection (1.5.0)
    remote:        Installing tilt (1.4.1)
    remote:        Installing sinatra (1.4.3)
    remote:        Using bundler (1.3.2)
    remote:        Your bundle is complete! It was installed into ./vendor/bundle
    remote:        Cleaning up the bundler cache.
    remote: -----> Discovering process types
    remote:        Default process types for Ruby/Rack -> rake, console, web
    remote: -----> Build complete!
    remote: -----> Deploying awwyeah ...
    remote: -----> Application deployed:
    remote:        http://awwyeah.deploy.eol.org
    remote:
    To git@deploy.eol.org:awwyeah
       ade6ea2..787cdaa  master -> master
#and let's see if the site is online
    ~/dev/awyeah (master) $ curl http://awyeah.deploy.eol.org
    oh hell yeah
###time?
    ~/dev/awyeah (master) $ date
    Thu Jun 13 16:18:59 EDT 2013

#let's check on that formatting:
    Writing inode tables:  4289/29809
plenty of time (also, ext4/4TB is kinda slow...)
