# WEB SOLUTION WITH WORDPRESS
---
## STEP 0
---
### LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS WEB SERVER and DATABASE SERVER
![instances](pbl6/instances.png)
## STEP 1
---
### Attach 3 EBS volumes each to Web Server and database server EC2 instances
![webvolume](pbl6/webservervolume.png)
![dbvolume](pbl6/webservervolume.png)
### CONNECT TO THE WEB SERVER INSTANCE AND CREATE A SINGLE LOGICAL VOLUME FOR THE 3 VOLUMES
```bash
# Inspect blockdevices attached to the server
lsblk

# Check mount points and free space
df -h

# Partition the 3 volumes as Linux LVM
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```

```bash
# Check the new configured partitions
lsblk

# Install lvm2
sudo yum install lvm2

# Check available
sudo lvmdiskscan

# Mark each of the 3 volumes as physcal volume to be used by LVM
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

# Verify the physical volumes has been created successfully
sudo pvs

# Add all Physical volumes to one Volume Group
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

# Verify the Volume Group has been created
sudo vgs

# Create two Logical Volumes from the Volume Group
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

# Verify the Logical Volumes have been created
sudo lvs

# Verify the entire Setup
sudo vgdisplay -v
sudo lsblk
```
![logicalvolume](pbl6/logicalvol.png)
![vgdisplay](pbl6/vgdisplay.png)
```bash
# Format the Logical volumes with ext4 FileSystem
sudo mkfs.ext4 /dev/webdata-vg/apps-lv
sudo mkfs.ext4 /dev/webdata-vg/logs-lv

# Create /var/www/html directory to store website files
sudo mkdir -p /var/www/html

# Create /home/recovery/logs directory to store backup of log data
sudo mkdir -p /home/recovery/logs

# Mount apps-lv on /var/www/html
sudo mount /dev/webdata-vg/apps-lv /var/www/html

# Backup /var/log/. to /home/recovery/logs using rsync
sudo rsync -av /var/log/. /home/recovery/logs

# Mount logs-lv on /var/log
sudo mount /dev/webdata-vg/logs-lv /var/log

# Restore log data
sudo rsync -av /home/recovery/logs/. /var/log
```
```bash
# Check the block id of the Logical volumes
sudo blkid

# Update the /etc/fstab file 
sudo vi /etc/fstab

# Test the configuration
sudo mount -a

# Restart the daemon
sudo systemctl daemon-reload

#Verify setup
df -h
```
![blkid](pbl6/blkid.png)
## STEP 2
> Repeat the steps for webserver but create a db-lv instead of apps-lv, and mount it to /db instead of /var/www/html and name the volume group db-vg instead webdata-vg

## STEP 3
### Install WordPress on the Web Server EC2
```bash
# Update the repository
sudo yum -y update

# Install wget, Apache and its dependencies
sudo yum -y install wget httpd php php-fpm php-mysqlnd php-json

# Start Apache
sudo systemctl enable httpd
sudo systemctl start httpd

# Install php and its dependencies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
![apache](pbl6/apache.png)
> Download Wordpress for web server
```bash
# Restart Apache 
sudo systemctl restart httpd

# Download WordPress
mkdir wordpress

cd wordpress

# download wordpress
sudo wget http://wordpress.org/latest.tar.gz

# Extract wordpress
sudo tar xzvf latest.tar.gz

```
### INSTALL MYSQL ON DB SERVER

```bash
# Update repository
sudo yum -y update

# Install MySQL server
sudo yum install -y mysql-server

# Start mysql
sudo systemctl enable mysqld

sudo systemctl restart mysqld

sudo systemctl status mysqld

# Setup mysql and create user
sudo mysql
```
![sqlserver](pbl6/sqlserver.png)
## SETUP WORDPRESS
```bash
# Setup wordpress
cd wordpress

sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php

# Copy wordpress files to the /var/www/html/ directory
sudo cp -R wordpress/. /var/www/html/

# Insert the database details
cd /var/www/html/

sudo vi wp-config.php

# Restart apache
sudo systemctl restart httpd
```
![wpconfig](pbl6/wpconfig.png)
### Install MySQL client and test that you can connect from your Web Server to your DB server
![sqlclient](pbl6/sqlclient.png)

### ACCESS WORDPRESS FROM BROWSER
![wordpress](pbl6/wordpress.png)