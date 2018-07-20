+++
title = "composer update is slow"
draft = false
date = "2018-07-20T13:09:20+09:00"
tags = ["php", "composer"]

+++

<!--more-->

## TL;DR

- Executing `compser update <package name>` on Vagrant is so slow.  
- Execute it on your local machine directly
- `compser update` seems comsuming a lot of computer resources like CPU or Memory.  

---

`compser update <package name>` is so slow when I execute it on Vagrant.  
This is caused because Vangrant has the limited computer resources like CPU or Memory.  
And `compser update` seems comsuming a lot of computer resources.  
<https://github.com/composer/composer/issues/1805>  

A workaround for this is executing it on your local machine directly, not on Vagrant.  
Because it should have more powerful cpu and bigger memory.  

### So slow on Vagrant

My Vagrant spec is;  

`Vagrantfile`  

```
...
config.vm.provider :virtualbox do |vb|
  vb.customize ["modifyvm", :id, "--memory", "512"]
  vb.customize ["modifyvm", :id, "--cpus", 2]
end
...
```
I added a swap in order to prevent the termination caused by Out of Memory.  

```
vagrant@sandbox:/srv/local/panel$ swapon -s
Filename				Type		Size	Used	Priority
/dev/mapper/precise64-swap_1            partition	786428	565820	-1
/swapfile                               file		16785404	0	-2
```

I executed the below command.  
```
vagrant@sandbox:/srv/local/panel$ composer --no-dev -vvv --profile update ringcaptcha/app-lookup
Reading ./composer.json
Loading config file ./composer.json
Checked CA file /etc/ssl/certs/ca-certificates.crt: valid
Executing command (/srv/local/panel): git branch --no-color --no-abbrev -v
Reading /home/vagrant/.composer/composer.json
Loading config file /home/vagrant/.composer/composer.json
Reading /srv/local/panel/vendor/composer/installed.json
Reading /home/vagrant/.composer/vendor/composer/installed.json
Loading plugin Hirak\Prestissimo\Plugin
Downloading https://packagist.org/packages.json
Writing /home/vagrant/.composer/cache/repo/https---packagist.org/packages.json into cache
    1/4:	http://packagist.org/p/provider-2018-07$b7d1972f4b5592672d3041c6fc96759395b5d6a233e3244c49ef1fa0aa89d0b0.json
    2/4:	http://packagist.org/p/provider-latest$672c0071fa698a90e507dd61adb926774c7c19bdaa1284b198861bc6cf4de78f.json

...

[567.5MB/2546.62s] Installing assets for Sensio\Bundle\DistributionBundle into web/bundles/sensiodistribution
[567.4MB/2546.77s] Memory usage: 567.42MB (peak: 2135.2MB), time: 2546.77s
```

It took `2546.77s`, 42 minutes.  

### Execute it on your local machine directly

My machine is Mac.  
So this is a workaround for Mac.  

- My Mac Specs  

```
macOS Sierra
Version 10.12.6 (16G1408)
MacBook Pro (13-inch, 2016, Two Thunderbolt 3 ports)
Processor 2 GHz Intel Core i5
Memory 8 GB 1867 MHz LPDDR3
```

- Requirements  
  - composer <https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx>  

I logged out Vagrant and executed the command in Vagrant synced folder on my Mac.  

```
yuta:~/dev/ringcaptcha/panel
$ COMPOSER_MEMORY_LIMIT=-1 php composer.phar --no-dev -vvv --profile update ringcaptcha/app-lookup
Reading ./composer.json
Loading config file ./composer.json

...

[568.5MB/200.31s] Installing assets as hard copies.
Installing assets for Symfony\Bundle\FrameworkBundle into web/bundles/framework
[568.5MB/200.32s] Installing assets for Sensio\Bundle\DistributionBundle into web/bundles/sensiodistribution
[568.5MB/200.38s] Memory usage: 568.45MB (peak: 2135.72MB), time: 200.38s
```

It took `200.38s`, less than 4 minutes.  

By the way, `COMPOSER_MEMORY_LIMIT=-1` in the command is for;  
<https://getcomposer.org/doc/articles/troubleshooting.md#memory-limit-errors>  
