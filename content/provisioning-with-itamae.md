+++
date = "2015-04-07T22:42:50+09:00"
draft = true
title = "provisioning with itamae"

+++

I provisioned my company's webhook server with [itamae](https://github.com/itamae-kitchen/itamae) recently.  
itamae is provisioning tool created by [cookpad](https://cookpad.com/en/categories/japanese-recipes) and formerly called as Lightchef (named after [Chef](https://www.chef.io/chef/)).  
In conclusion, it is great tool because of it's simplicity.  
It is very easy to start provisioning with itamae.  
<!--more-->

## My first provisioning tool

I've been using [Docker](https://www.docker.com/) to create vm machines for develop.  
But my company's webhook server that I was going to build had to run as normal server, not as container.  
So I decided to build it with some provisioning tool.  

I started with searching tools because I have never used provisioning tools like [Chef](https://www.chef.io/chef/) or [Ansible](http://www.ansible.com/).  

Chef seemed very complicatied and taking much time to start.  
So I don't want to use it.  
I had been thinking Ansible had suited my occasion better until I found itamae.  

itamae has good DSL like `not_if`.  
Ansible configuration is to be defined by yaml, So it is weaker than itamae in setting flexibility.  
itamae is also influenced very much by Chef  however it is much simpler than Chef instead of .  

Actually it didn't cost me much time to learn and go.  
