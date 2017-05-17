+++
date = "2017-05-17T12:19:16+09:00"
title = "Xdebug remote cli debugging with vagrant and PhpStorm"
draft = false
tags = ["vagrant", "php", "xdebug", "phpstorm"]

+++

<!--more-->

There are three points to set xdebug's remote cli debugging correctly.  
- [php.ini for cli](#php-ini-for-cli)  
- [PHP_IDE_CONFIG environment variable](#php-ide-config-environment-variable)  
- [Phpstorm setting](#phpstorm-setting)  

## php.ini for cli

`xdebug.remote_host` means the host where phpstorm runs.(= client)  
You should specify the host ip from the guest machine, vagrant.  
You can use `10.0.2.2` as it.  

```
zend_extension = /usr/lib/php5/20100525/xdebug.so
xdebug.remote_enable  = on
xdebug.remote_autostart = 1
xdebug.remote_host = 10.0.2.2
xdebug.remote_port=9000
xdebug.remote_handler = dbgp
xdebug.var_display_max_depth = -1
xdebug.var_display_max_children = -1
xdebug.var_display_max_data = -1
xdebug.idekey = PHPSTORM
```

### (TL;DR)
You will see below when you hit `ifconfig` command in vagrant's shell.  
```
eth0      Link encap:Ethernet  HWaddr 08:00:27:88:0c:a6  
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe88:ca6/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:267539 errors:0 dropped:0 overruns:0 frame:0
          TX packets:154775 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:259150497 (259.1 MB)  TX bytes:26565361 (26.5 MB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:10:94:c7  
          inet addr:192.168.55.2  Bcast:192.168.55.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe10:94c7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:28705 errors:0 dropped:0 overruns:0 frame:0
          TX packets:21756 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:2879409 (2.8 MB)  TX bytes:21317169 (21.3 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:4853 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4853 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:948041 (948.0 KB)  TX bytes:948041 (948.0 KB)
```

`10.0.2.2` is not there, but you can spcify the host of vm with it.  
<http://stackoverflow.com/questions/33777041/why-10-0-2-2-was-not-there-with-running-ifconfig>  
  
"10.0.2.2 always points to the local host when you are running emulator or vm. So in virtual machine , it refers to the local host (127.0.0.1) as 10.0.2.2. That is the reason you can't see it in ifconfig in your host."  

## PHP_IDE_CONFIG environment variable

You need to set this for phpstorm's path mapping.  
  
"To tell PhpStorm which path mapping configuration it should use for a connection from a certain machine you need to set the value of the “PHP_IDE_CONFIG” environment variable to “serverName=SomeName”, where ‘SomeName’ is the name of the server configured in ‘Project Settings | PHP | Servers’".  
  
From <https://blog.jetbrains.com/phpstorm/2012/03/new-in-4-0-easier-debugging-of-remote-php-command-line-scripts/>  

My application uses supervisor to execute php cli command, so I passed the environment variable to supervisor.  

supervisor config file  
```
[program:my_worker]
directory={{API_PATH}}
command={{API_PATH}}/my_worker.php "verificationWorker.php"
environment=PHP_IDE_CONFIG="serverName=vagrant"
user=www-data
autostart=false
autorestart=true
```

`serverName` value should correspond to phpstorm's server value in `Project Settings | PHP | Servers`.  
See below.  

## Phpstorm setting

Specify `Name` and `Path mappings` correctly.  
`Host` and `Port` values don't affect anything. You can set them as anything.  
This setting is just for relating `serverName` in PHP_IDE_CONFIG with `Path mappings`.  

<a href="/images/xdebug-remote-cli-debug-with-vagrant-01.png"><img src="/images/xdebug-remote-cli-debug-with-vagrant-01.png" class="image"></a>  

References:  
<https://blog.jetbrains.com/phpstorm/2012/03/new-in-4-0-easier-debugging-of-remote-php-command-line-scripts/>  
<http://stackoverflow.com/questions/19917148/tell-vagrant-the-ip-of-the-host-machine>  
<https://xdebug.org/docs/remote>  
<http://www.1x1.jp/blog/2014/08/how-to-setup-php-remote-debug-with-vagrant-vm.html>
