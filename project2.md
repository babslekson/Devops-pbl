# LEMP STACK INSTALLATION
---
## STEP 0
---
### Create and launch EC2 instance
![instance](pbl2/instance.png)
## Installing NGINX server
```bash
sudo apt update
sudo apt install nginx

#verify nginx server
sudo systemctl status nginx
```
![nginx](pbl2/nginx.png)

## NGINX on instance URL
![nginx](pbl2/nginx_url.png)
## STEP 2
### Installing mysql
```bash
sudo apt install mysql-server
# Setup
sudo mysql_secure_installation

sudo mysql
```

## STEP 3
---
### Installing PHP
```bash
sudo apt install php-fpm php-mysql
```
## STEP 4
---
### Configure NGINX to use PHP processor
```bash
sudo mkdir /var/www/projectLEMP

#change ownership
sudo chown -R ubuntu:ubuntu /var/www/projectLEMP

# New config file in nginx's sites-available
sudo vim /etc/nginx/sites-available/projectLEMP
```
content of config
```bash
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
```bash
# Activate configuration by linking to the config file from Nginx’s sites-enabled directory
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

# Test config file for syntax errors
sudo nginx -t

# Disable nginx default host
sudo unlink /etc/nginx/sites-enabled/default

# Restart nginx
sudo systemctl reload nginx

# Create index.html file in the /var/www/projectlemp directory
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
![nginx_config](pbl2/nginx_config.png)
>Open website URL using IP address

http://\<Public-Ip_Address>:80
![nginx_echo](pbl2/nginx_url_echo.png)
