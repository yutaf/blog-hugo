+++
title = "phpbrew note2"
draft = false
date = "2018-07-20T13:09:19+09:00"
tags = ["php", "phpbrew"]

+++

<!--more-->

This is a note for building php by phpbrew on Mac.  

- Restore icu4c to the old version to build php  
```
$ cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula
$ git checkout ab72fee8166749231036a4476aaba30df9619800 icu4c.rb
$ brew unlink icu4c
$ HOMEBREW_NO_AUTO_UPDATE=1 brew reinstall icu4c
```
References;  
<https://qiita.com/sizuhiko/items/aebdbb05f0824cae4bdd>  
<https://github.com/php-build/php-build/pull/463>  
<https://qiita.com/KyoheiG3/items/912bcc27462871487845>  


- Install other dependencies to build php  
```
$ brew install \
  libxml2 \
  jpeg \
  gd \
  mcrypt
```

- Setting up Configuration for phpbrew  
~/local/etc/phpbrew.yaml

```
variants:
  dev:
    mbstring:
    intl:
    icu:
    gettext:
    pcre:
    readline:
    xml:
    soap:
    mysql:
    my-without-pdo-sqlite:
      - --without-pdo-sqlite
    my-without-sqlite3:
      - --without-sqlite3
    zlib:
    openssl:
    mcrypt:
    bcmath:
    curl:
    zip:
    my-disable-cgi:
      - --disable-cgi
    calendar:
    my-disable-short-tags:
      - --disable-short-tags
```

```
$ phpbrew init -c ~/local/etc/phpbrew.yaml
```

References;  
<https://github.com/phpbrew/phpbrew/wiki/Setting-up-Configuration>  

- Dryrun  
```
phpbrew -d install -d 5.4.45 +neutral +dev
```

- Install  
```
phpbrew -d install --stdout 5.4.45 +neutral +dev
```

- Switch php version  

```
$ phpbrew list
* (system)
  php-5.4.45
$ phpbrew use php-5.4.45
$ php -v
PHP 5.4.45 (cli) (built: Jul 20 2018 12:46:03)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2014 Zend Technologies
```

- Set php timezone to avoid Exception  

```
$ php -i | grep Loaded
Loaded Configuration File => /Users/yuta/.phpbrew/php/php-5.4.45/etc/php.ini
```

```
$ vim /Users/yuta/.phpbrew/php/php-5.4.45/etc/php.ini
```
Update this part  
```
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
;date.timezone =
```
To  
```
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = UTC

```
