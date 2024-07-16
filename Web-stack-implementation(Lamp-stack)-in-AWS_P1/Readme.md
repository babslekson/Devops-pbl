
# LAMP STACK IMPLEMENTATION
---
## STEP 0
---
### Create an AWS account
![aws_account](images/aws_account.png)

### Create and launch an EC2 instance
![instance](images/instance.png)


### Connect to instance
![connect_instance](images/connect_instance.png)


## STEP 1 
---
### Installing Apache
```bash
sudo apt update 
sudo apt install apache2

# Verifying that apache is installed
sudo systemctl status apache2
```
![apache](images/apache.png)
### Apache on instance URL
>http://\<Public-IP-Address\>:80

![APACHE_URL](images/apache_url.png)

## STEP 2
---
### Installing Mysql
```bash
sudo apt install mysql-server
sudo mysql
#remove insure defsult settings
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<PassWord.1>';

#setup user
sudo mysql_secure_installation
```

![sql](images/sql.png)

## STEP 3
---
### Installing PHP
```bash
sudo apt install php libapache2-mod-php php-mysql

#verify that php is installed
php -v
```

![php](images/php.png)

## STEP 4
---
CREATING A VIRTUAL HOST FOR THE WEBSITE USING APACHE

```bash
sudo mkdir /var/www/projectlamp

#change ownership
sudo chown -R ubuntu:ubuntu /var/www/projectlamp
# write into config file
sudo vim /etc/apache2/sites-available/projectlamp
```
>content of projectlamp.conf
```bash
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
```bash
# enable virtual host 
sudo a2ensite projectlamp

#disable default website
sudo a2dissite 000-default

#check for configuration error
sudo apache2ctl configtest

# Create index.html file in the /var/www/projectlamp directory
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
![virtual_host](images/virtual_host.png)

### Open website using ip address
>http://\<Public-Ip-Address>:80

![virtual_host_url](images/virtual_host_url.png)

## STEP 5
### Enable PHP on the website
```bash
# Changing the files' order of precedence
sudo vim /etc/apache2/mods-enabled/dir.conf
```
```bash
<IfModule mod_dir.c>
#Change this:
#DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
#To this:
DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
```bash
# Restart Apache
sudo systemctl reload apache2

# Create index.php file
vim /var/www/projectlamp/index.php
```
```bash
<?php
phpinfo();
```
![php_info](images/php_info.png)

### DONE!!