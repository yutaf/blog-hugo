+++
date = "2015-04-07T22:42:50+09:00"
draft = false
title = "provisioning with itamae"
tags = ["provisioning", "itamae"]

+++

I provisioned my company's webhook server with [itamae](https://github.com/itamae-kitchen/itamae) recently.  
itamae is a *very simple* provisioning tool created by [cookpad](https://cookpad.com/en/categories/japanese-recipes) and formerly they called it as Lightchef.  
<!--more-->

I have never used Chef and also other provisioning tools like Ansible.  
But I could learn it for 2,3 hours and began to provision.  
It is very easy to start provisioning with itamae.  
I like it.  

## How to use it

First of all we need `recipe` to provision.  
Here's my recipe to install ruby with rbenv.  

`recipes/ruby.rb`

```
execute "install rbenv" do
  command "
    git clone https://github.com/sstephenson/rbenv.git ~/.rbenv && \
    echo 'export PATH=\"$HOME/.rbenv/bin:$PATH\"' >> ~/.bashrc  && \
    echo 'eval \"$(rbenv init -)\"' >> ~/.bashrc && \
    git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
  "
  not_if "test -d ~/.rbenv"
end
execute "install ruby" do
  command "rbenv install 2.2.1 && rbenv global 2.2.1"
  not_if "test -d /root/.rbenv/versions/2.2.1"
end
```

Then execute this recipe via ssh.  

```
itamae ssh -h 54.64.46.48 -u webhook recipes/ruby.rb
```

## Details

#### About recipe

- We don't need to write `sudo` specifically because itamae automatically does it well.  
- `not_if` is very useful [resource](https://github.com/itamae-kitchen/itamae/wiki/Resources).  
With that we can avoid unnecessary execution of recipes when we do it repeatedly.  

#### Execution

- we can of course execute itamae locally not via ssh.  

## More to it

#### Including Recipes

Paths are indicated relatively from the recipe.  

```
include_recipe "yum.rb"
include_recipe "apache22-php.rb"
include_recipe "ruby.rb"
include_recipe "gem.rb"
include_recipe "npm.rb"
include_recipe "ryogoku.rb"
include_recipe "add_deploy_user.rb"
include_recipe "add_webhook_script.rb"
include_recipe "cron.rb"
```

#### More recipes

`recipes/add_deploy_user.rb`

```
execute "add deploy user" do
  # you cannot ssh unless changing password
  command "
    adduser --system --shell /bin/bash --home-dir /home/deploy --user-group deploy && \
    passwd -f -u deploy && \
    mkdir -p /home/deploy/.ssh
  "
  not_if "grep ^deploy /etc/passwd"
end
# Set up key
remote_directory "/home/deploy/.ssh" do
  source "../templates/.ssh/"
end
execute "Change ssh files permission" do
  command "
    chown -R deploy:deploy /home/deploy && \
    chmod 700 /home/deploy/.ssh && \
    chmod 600 /home/deploy/.ssh/authorized_keys && \
    chmod 600 /home/deploy/.ssh/id_rsa
  "
end
execute "disable requiretty" do
  command "sed -i 's;^Defaults *requiretty;#&;g' /etc/sudoers"
end
execute "enable apache user to execute ryogoku without password" do
  command "echo 'apache  ALL=(deploy) NOPASSWD: /usr/local/bin/ryogoku' >> /etc/sudoers"
  not_if "grep ^apache /etc/sudoers"
end
```

`recipes/apache22-php.rb`

```
execute "mkdir -p /usr/local/src" do
  command "mkdir -p /usr/local/src"
  not_if "test -d /usr/local/src"
end

#
# apache
#

execute "download apache tarball" do
  command "cd /usr/local/src && curl -L -O http://apache.cs.utah.edu//httpd/httpd-2.2.29.tar.gz"
  not_if "test -e /usr/local/src/httpd-2.2.29.tar.gz"
end

execute "tar apache tarball" do
  command "cd /usr/local/src && tar xzvf httpd-2.2.29.tar.gz"
  not_if "test -d /usr/local/src/httpd-2.2.29"
end

execute "install apache" do
  command "
  cd /usr/local/src/httpd-2.2.29 && \
  ./configure \
    --prefix=/opt/apache2.2.29 \
    --enable-mods-shared=all \
    --enable-proxy \
    --enable-ssl \
    --with-ssl \
    --with-mpm=prefork \
    --with-pcre && \
  make && \
  make install
  "
  not_if "test -d /opt/apache2.2.29"
end

# Apache config
sed_scripts_for_httpd_conf = [
  's/^Listen 80/#&/',
  's/^DocumentRoot/#&/',
  '/^<Directory/,/^<\/Directory/s/^/#/',
  's;ScriptAlias /cgi-bin;#&;',
  's;#\(Include conf/extra/httpd-mpm.conf\);\1;',
  's;#\(Include conf/extra/httpd-default.conf\);\1;',
  '/^\s*DirectoryIndex/s/$/ index.php/', # DirectoryIndex; index.html precedes index.php
  # Change User & Group
  's;^\(User \)daemon$;\1apache;',
  's;^\(Group \)daemon$;\1apache;'
]

sed_scripts_for_httpd_conf.each do |sed_script_for_httpd_conf|
  execute sed_script_for_httpd_conf do
    command 'sed -i \'' + sed_script_for_httpd_conf + '\' /opt/apache2.2.29/conf/httpd.conf'
  end
end

execute "edit httpd.conf" do
  command 'echo "Include /srv/apache/apache.conf" >> /opt/apache2.2.29/conf/httpd.conf'
  not_if "grep /srv/apache/apache.conf /opt/apache2.2.29/conf/httpd.conf"
end

execute "edit httpd-defaut.conf" do
  command 'sed -i "s/\(ServerTokens \)Full/\1Prod/" /opt/apache2.2.29/conf/extra/httpd-default.conf'
end

# Change User & Group
#  useradd --system --shell /usr/sbin/nologin --user-group --home /dev/null apache; \
execute "add apache user & group" do
  command "useradd --system --shell /usr/sbin/nologin --user-group --home /dev/null apache"
  not_if "grep '^apache' /etc/passwd"
end

execute "mkdir -p /srv/apache" do
  command "mkdir -p /srv/apache"
  not_if "test -d /srv/apache"
end

remote_file "/srv/apache/apache.conf" do
  source "../templates/apache.conf"
end

execute "set apache CustomLog" do
  command 'echo \'CustomLog "|/opt/apache2.2.29/bin/rotatelogs /srv/www/logs/access/access.%Y%m%d.log 86400 540" combined\' >> /srv/apache/apache.conf'
  not_if "grep '^CustomLog' /srv/apache/apache.conf"
end

execute "set apache ErrorLog" do
  command 'echo \'ErrorLog "|/opt/apache2.2.29/bin/rotatelogs /srv/www/logs/error/error.%Y%m%d.log 86400 540"\' >> /srv/apache/apache.conf'
  not_if "grep '^ErrorLog' /srv/apache/apache.conf"
end

execute "make log dirs" do
  command "mkdir -m 777 -p /srv/www/logs/{access,error,app}"
  not_if "test -d /srv/www/logs/access"
end

execute "make Apache document root" do
  command "mkdir -p /srv/www/htdocs/"
  not_if "test -d /srv/www/htdocs/"
end

#
# php
#

execute "download php tarball" do
  command "cd /usr/local/src && curl -L -O http://php.net/distributions/php-5.6.7.tar.gz"
  not_if "test -e /usr/local/src/php-5.6.7.tar.gz"
end

execute "tar php tarball" do
  command "cd /usr/local/src && tar xzvf php-5.6.7.tar.gz"
  not_if "test -d /usr/local/src/php-5.6.7"
end

execute "install php" do
  command "
  cd /usr/local/src/php-5.6.7 && \
  ./configure \
    --prefix=/opt/php-5.6.7 \
    --with-config-file-path=/srv/php \
    --with-apxs2=/opt/apache2.2.29/bin/apxs \
    --with-libdir=lib64 \
    --enable-mbstring \
    --enable-intl \
    --with-icu-dir=/usr \
    --with-gettext=/usr \
    --with-pcre-regex=/usr \
    --with-pcre-dir=/usr \
    --with-readline=/usr \
    --with-libxml-dir=/usr/bin/xml2-config \
    --with-zlib=/usr \
    --with-zlib-dir=/usr \
    --with-gd \
    --with-jpeg-dir=/usr \
    --with-png-dir=/usr \
    --with-freetype-dir=/usr \
    --enable-gd-native-ttf \
    --enable-gd-jis-conv \
    --with-openssl=/usr \
    --with-mcrypt=/usr \
    --enable-bcmath \
    --with-curl \
    --enable-exif && \
  make && \
  make install
  "
  not_if "test -d /opt/php-5.6.7"
end

#
# xdebug
#

# set php PATH because using phpize for xdebug installation
execute "set php PATH" do
  command "echo 'export PATH=/opt/php-5.6.7/bin:$PATH' >> ~/.bashrc"
  not_if "grep /opt/php-5.6.7/bin ~/.bashrc"
end

execute "install xdebug" do
  command "
    cd /usr/local/src && \
    curl -L -O http://xdebug.org/files/xdebug-2.3.2.tgz && \
    tar -xzf xdebug-2.3.2.tgz && \
    cd xdebug-2.3.2 && \
    phpize && \
    ./configure --enable-xdebug && \
    make && \
    make install
  "
  not_if "test -e /opt/php-5.6.7/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so"
end

# php.ini
execute "mkdir -p /srv/php" do
  command "mkdir -p /srv/php"
  not_if "test -d /srv/php"
end
remote_file "/srv/php/php.ini" do
  source "../templates/php.ini"
end
execute "Add zend_extension directive" do
  command "echo 'zend_extension = \"/opt/php-5.6.7/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so\"' >> /srv/php/php.ini"
  not_if "grep ^zend_extension /srv/php/php.ini"
end


execute "start apache" do
  command "/opt/apache2.2.29/bin/apachectl start"
  not_if "ps aux | grep httpd | grep -v grep"
end
```
