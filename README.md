#!/bin/bash

DB_NAME="database-1-beta"
DB_USERNAME="admin"
DB_PASSWORD="Beta12345"
DB_HOST="database-1-beta-instance-1.czs0uqeyypcc.eu-west-1.rds.amazonaws.com"
EFS_FS_ID="fs-067e64d854c3565b1"

dnf update -y

#install wget, apache server, php and efs utils
dnf install -y httpd wget php-fpm php-mysqli php-json php amazon-efs-utils

#create wp-content mountpoint
mkdir -p /var/www/html/wp-content
mount -t efs $EFS_FS_ID:/ /var/www/html/wp-content

#install wordpress
cd /var/www
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
rm -f latest.tar.gz

#change wp-config with DB details
cp -rn wordpress/* /var/www/html/
sed -i "s/database_name_here/$DB_NAME/g" /var/www/html/wp-config.php
sed -i "s/username_here/$DB_USERNAME/g" /var/www/html/wp-config.php
sed -i "s/password_here/$DB_PASSWORD/g" /var/www/html/wp-config.php
sed -i "s/localhost/$DB_HOST/g" /var/www/html/wp-config.php

#change httpd.conf file to allowoverride
#  enable .htaccess files in Apache config using sed command
sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# create phpinfo file
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

# Recursively change OWNER of directory /var/www and all its contents
chown -R apache:apache /var/www

systemctl restart httpd
systemctl enable httpd
