# phpmyadmin-apache-mysql-connection

https://gist.github.com/kitloong/5f66a140f38b9698e8c3ed13b968ff47

# !!! This guide was created with Macbook Pro M1. Path may vary for different or even same machine.

# Install Homebrew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

# Install PHP

```
brew install php
```
```
To enable PHP in Apache add the following to httpd.conf and restart Apache:
LoadModule php_module /opt/homebrew/opt/php/lib/httpd/modules/libphp.so

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

Finally, check DirectoryIndex includes index.php
    DirectoryIndex index.php index.html

The php.ini and php-fpm.ini file can be found in:
    /opt/homebrew/etc/php/8.1/

To restart php after an upgrade:
  brew services restart php
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/php/sbin/php-fpm --nodaemonize
```
``` 
brew services restart php
```
```
php --version
PHP 8.1.2 (cli) (built: Jan 21 2022 04:34:01) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2, Copyright (c), by Zend Technologies
```
    
## Update config
    
```
php -i | grep "additional .ini"
Scan this dir for additional .ini files => /opt/homebrew/etc/php/8.1/conf.d

# Instead of modify setting in default .ini file, /opt/homebrew/etc/php/8.1/php.ini
# You should vi new .ini file to /opt/homebrew/etc/php/8.1/conf.d
# All .ini files in conf.d will be scanned and loaded

# Example
echo "memory_limit = 512M" > /opt/homebrew/etc/php/8.1/conf.d/memory-limit.ini

php --ini
Configuration File (php.ini) Path: /opt/homebrew/etc/php/8.1
Loaded Configuration File:         /opt/homebrew/etc/php/8.1/php.ini
Scan for additional .ini files in: /opt/homebrew/etc/php/8.1/conf.d
Additional .ini files parsed:      /opt/homebrew/etc/php/8.1/conf.d/ext-opcache.ini,
/opt/homebrew/etc/php/8.1/conf.d/memory-limit.ini

php -i | grep "memory_limit"               
memory_limit => 512M => 512M
```

# Install Apache

```
brew install httpd
```
```
DocumentRoot is /opt/homebrew/var/www.

The default ports have been set in /opt/homebrew/etc/httpd/httpd.conf to 8080 and in
/opt/homebrew/etc/httpd/extra/httpd-ssl.conf to 8443 so that httpd can run without sudo.

To restart httpd after an upgrade:
  brew services restart httpd
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/httpd/bin/httpd -D FOREGROUND
```
```
brew services restart httpd
```
```
httpd -v
Server version: Apache/2.4.52 (Unix)
Server built:   Dec 20 2021 13:35:09
```
    
## Create httpd-php.conf

```
vi /opt/homebrew/etc/httpd/extra/httpd-php.conf

# Insert following
DirectoryIndex index.php index.html # Add index.php

# php-fpm default port
<FilesMatch \.php$>
    SetHandler "proxy:fcgi://127.0.0.1:9000"
</FilesMatch>
```

## Update Apache config

```
vi /opt/homebrew/etc/httpd/httpd.conf

# Change listen to port 8080
Listen 8080 # Port 80 common used by apps like Skype or Teams, change to other port such as 8080 for convenience.

# Enable following modules by uncomment
LoadModule proxy_module lib/httpd/modules/mod_proxy.so
LoadModule proxy_fcgi_module lib/httpd/modules/mod_proxy_fcgi.so
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so

# Append at the end of the conf
# Load php fcgi configuration
Include /opt/homebrew/etc/httpd/extra/httpd-php.conf
brew services start httpd
```

## Test
    
```
echo "<?php echo phpinfo();" > /opt/homebrew/var/www/info.php

curl -I http://localhost:8080/info.php
```

# Install MySQL@5.7

```
brew install mysql@5.7
```
```
We've installed your MySQL database without a root password. To secure it run:
mysql_secure_installation

MySQL is configured to only allow connections from localhost by default

To connect run:
    mysql -uroot

mysql@5.7 is keg-only, which means it was not symlinked into /opt/homebrew,
because this is an alternate version of another formula.

If you need to have mysql@5.7 first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/mysql@5.7/bin:$PATH"' >> ~/.zshrc

For compilers to find mysql@5.7 you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/mysql@5.7/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/mysql@5.7/include"


To restart mysql@5.7 after an upgrade:
  brew services restart mysql@5.7
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/mysql@5.7/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql

# Backup existing mysql localtion if needed
ls -la $(which mysql)     
lrwxr-xr-x  1 liow.kitloong  admin  36 Mar 11  2019 /usr/local/bin/mysql -> ../Cellar/mysql@5.7/5.7.25/bin/mysql
```
```
# Link mysql to mysql@5.7
brew link mysql@5.7 --force

brew services restart mysql@5.7
```
```
mysql -V
mysql  Ver 14.14 Distrib 5.7.37, for osx10.16 (x86_64) using  EditLine wrapper
```

# Install phpMyAdmin

```
brew install phpmyadmin
```
```
To enable phpMyAdmin in Apache, add the following to httpd.conf and
restart Apache:
    Alias /phpmyadmin /opt/homebrew/share/phpmyadmin
    <Directory /opt/homebrew/share/phpmyadmin/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        <IfModule mod_authz_core.c>
            Require all granted
        </IfModule>
        <IfModule !mod_authz_core.c>
            Order allow,deny
            Allow from all
        </IfModule>
    </Directory>
Then open http://localhost/phpmyadmin
The configuration file is /opt/homebrew/etc/phpmyadmin.config.inc.php
```

## Update config

```
vi /opt/homebrew/etc/phpmyadmin.config.inc.php
        
# Extend cookies lifetime
$cfg['LoginCookieValidity'] = 7*24*60*60;
```

## Create phpMyAdmin.conf   

```
vi /opt/homebrew/etc/httpd/extra/phpmyadmin.conf

# Insert following
Alias /phpmyadmin /opt/homebrew/share/phpmyadmin
<Directory /opt/homebrew/share/phpmyadmin/>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    <IfModule mod_authz_core.c>
        Require all granted
    </IfModule>
    <IfModule !mod_authz_core.c>
        Order allow,deny
        Allow from all
    </IfModule>
</Directory>
```

## Load phpMyAdmin.conf in Apache
 
```
vi /opt/homebrew/etc/httpd/httpd.conf
# Append at the end of the conf
# Load phpMyAdmin configuration
Include /opt/homebrew/etc/httpd/extra/phpmyadmin.conf
```

Restart httpd

```
brew services restart httpd
```

## Validate

```
curl -I http://localhost:8080/phpmyadmin
```

## Multi Hosts

```
vi /opt/homebrew/etc/phpmyadmin/config.inc.php

# Add
$i++;
$cfg['Servers'][$i]['verbose'] = 'Server Name';
$cfg['Servers'][$i]['host'] = '128.0.0.2'; // New host
$cfg['Servers'][$i]['port'] = 3306;
$cfg['Servers'][$i]['socket'] = '';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['extension'] = 'mysqli';
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['AllowNoPassword'] = false;
```

## Prefix credential in config (use at own risk!)

```
vi /opt/homebrew/etc/phpmyadmin/config.inc.php

$cfg['Servers'][$i]['auth_type'] = 'config'; # Use config

$cfg['Servers'][$i]['user'] = 'username';
$cfg['Servers'][$i]['password'] = 'password';
```
