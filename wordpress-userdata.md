# READ THESE FOLLOWING INSTRUCTION BEFORE COPYING THESE DATA
# for wordpress data, we need to update the mount point to our file system by following the step below
#  go to EFS > Access point > select wordpress >  View details > Attach 
# copy efs mount helper without last efs part and paste it
# between  "sudo mount -t efs -o tls, ..........  :/ /var/www/" on line 19
#
# we are creating healthstatus file (empty) so the loadbalancer will see our instance as healthy
# we need to change our rds endpoint
# go to rds > db instances
# click on acs-database
# copy the endpoint
# paste it between "sed -i "s/localhost/.........../g" wp-config.php " on line 36
# note that we have not created wordpressdb on line 39, but we are going to create it



#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-05cde16db4212f446 fs-07d12dccdc93fa0bd:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/acs-database.cxgqmm0me4bb.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
