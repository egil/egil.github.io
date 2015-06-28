---
layout: post
title: 'Drupal/PHP: Setting up a Developing Environment - Windows 7 and Ubuntu Server
  in virtual machine'
created: 1317604088
---
## VMWare
- NAT
- Disable all hardware not needed on server (floppy, soundcard, etc.)
## Ubuntu Server
- Install using standard settings (keyboard, full harddrive)
- Make sure to choose the following during "Software selection"
  - OpenSSH
- Static IP
  - http://www.cyberciti.biz/tips/howto-ubuntu-linux-convert-dhcp-network-configuration-to-static-ip-configuration.html
- dnsmasq
  - uncomment: 
    - no-resolv
    - domain-needed
    - bogus-priv
    - address=/.dev/192.168.247.3
    - no-dhcp-interface= 
- Samba for file sharing

### LAMP
http://vmirgorod.name/11/1/20/drupal-development-environment-based-ubuntu-1010

#### PHPMYADMIN

~~~
# phpMyAdmin default Apache configuration

#Alias /phpmyadmin /usr/share/phpmyadmin
<VirtualHost *:80>
        DocumentRoot /usr/share/phpmyadmin
        ServerName pma.dev

        <Directory /usr/share/phpmyadmin>
                Options FollowSymLinks
                DirectoryIndex index.php

                <IfModule mod_php5.c>
                        AddType application/x-httpd-php .php

                        php_flag magic_quotes_gpc Off
                        php_flag track_vars On
                        php_flag register_globals Off
                        php_admin_flag allow_url_fopen Off
                        php_value include_path .
                        php_admin_value upload_tmp_dir /var/lib/phpmyadmin/tmp
                        php_admin_value open_basedir /usr/share/phpmyadmin/:/etc/phpmyadmin/:/var/lib/phpmyadmin/
                </IfModule>

        </Directory>
</VirtualHost>

# Authorize for setup
<Directory /usr/share/phpmyadmin/setup>
    <IfModule mod_authn_file.c>
    AuthType Basic
    AuthName "phpMyAdmin Setup"
    AuthUserFile /etc/phpmyadmin/htpasswd.setup
    </IfModule>
    Require valid-user
</Directory>

# Disallow web access to directories that don't need it
<Directory /usr/share/phpmyadmin/libraries>
    Order Deny,Allow
    Deny from All
</Directory>
<Directory /usr/share/phpmyadmin/setup/lib>
    Order Deny,Allow
    Deny from All
</Directory>
~~~

## Windows 7
 - Add Ubuntu Server ip to DNS for VMnet8
