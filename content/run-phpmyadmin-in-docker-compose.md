+++
date = "2016-10-18T18:15:41+09:00"
draft = false
title = "Run phpmyadmin in docker-compose"
tags = ["docker", "mysql"]

+++

<!--more-->

<https://hub.docker.com/r/phpmyadmin/phpmyadmin/>  

Set `PMA_HOST`, `PMA_PORT` as environment of phpmyadmin in docker-compose.yml to connect separeted mysql container.  

docker-compose.yml  

```
mysql:
  build: mysql
  environment:
    MYSQL_USER: dbuser
    MYSQL_PASSWORD: dbpassword
    MYSQL_ROOT_PASSWORD: dbrootpassword
    MYSQL_DATABASE: dbdatabase
  volumes_from:
    - datastore
phpmyadmin:
  image: phpmyadmin/phpmyadmin
  links:
    - mysql
  environment:
    PMA_HOST: mysql
    PMA_PORT: 3306
  ports:
    - '8080:80'
datastore:
  image: busybox
  volumes:
    - /var/lib/mysql
php:
  build: php
  ports:
    - '80:80'
  links:
    - mysql
  volumes:
    - ./php/www:/srv/www
```
