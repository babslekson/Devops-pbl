
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
