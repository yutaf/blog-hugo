+++
date = "2017-06-10T21:15:28+09:00"
draft = false
title = "supervisord's PATH"
tags = ["supervisor"]

+++

<!--more-->

## Environment
- OS: Ubuntu 12.04 LTS  
- supervisor version: 3.0a8-1.1  

## What happned
When I executed php programme through supervisor, I got an error log below.  

supervisor config file  
```
...

[program:worker_verification]
directory=/srv/local/api
command=/srv/local/api/rcWorkerManager.php "rcVerificationWorker.php" "rabbit_queue" --env=prod
environment=PARAM5="dev",PHP_IDE_CONFIG="serverName=vagrant"
user=www-data
autostart=false
autorestart=true

...
```

log  
```
/usr/bin/env: php: No such file or directory
```

But php was already installed by source compile and I set a PATH to `/etc/environment` like below.  

```
PATH="/opt/php-5.4.45/bin:/opt/php-5.4.45/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"
```

So I checked the PATH value by showing it through supervisor, too.  
This is the part of the supervisor setting file to show PATH.

```
[program:show_path]
command=printenv
user=www-data
autostart=false
autorestart=true
```

Then I got log like below.  

```
SUPERVISOR_GROUP_NAME=show_path
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

PATH value was different from the previous one.  
Then I investigated which file defined this PATH value.  

```
vagrant@sandbox:/$ sudo find /etc -type f -exec grep -l '/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin' {} \;
/etc/crontab
/etc/init.d/unattended-upgrades
/etc/init.d/networking
/etc/init.d/nginx
/etc/init.d/supervisor
```

I found `/etc/init.d/supervisor`.  
Take a closer look at it.  

```
...

### BEGIN INIT INFO
# Provides:          supervisor
# Required-Start:    $remote_fs $network $named
# Required-Stop:     $remote_fs $network $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/stop supervisor
# Description:       Start/stop supervisor daemon and its configured
#                    subprocesses.
### END INIT INFO


PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/bin/supervisord
NAME=supervisord
DESC=supervisor

test -x $DAEMON || exit 0

...
```

This file specified PATH value as  
`PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`.  
Maybe this was why I got the PATH when I executed supervisor.  

## Workaround

This version of supervisor(3.0a8-1.1) didn't support to append paths to PATH unfortunately.  
<https://serverfault.com/questions/331027/supervisord-how-to-append-to-path>  
<https://github.com/Supervisor/supervisor/issues/599>  

So I decided to locate php executables to `/usr/local/bin` where supervisord could see.  

```
./configure \
  --prefix=/opt/php-5.4.45 \
...
  --bindir=/usr/local/bin \
  --sbindir=/usr/local/bin \
...
```
