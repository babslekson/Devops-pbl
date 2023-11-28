# AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology
---
## Set Up Architecture
![two company website architecture](pbl15/architecture.png)
---
### Create VPC 
![vpc](pbl15/vpc.png)
---
### Create public and private subnet
![subnets](pbl15/subnets.png)
---
### Create routing table for public and private subnets 
![routing table](pbl15/routetable.png)
---
### Create internet gateway and associate it with the public subnet
![internet gateway](pbl15/igw.png)
----
### Create a NAT gateway
![nat gateway](pbl15/nat.png)
---
### create security group for the nginx server, bastion server, webservers, application load balancers and data layer
![security group](pbl15/securitygroups.png)
---
### create EC2 instances for nginx, bastion and webservers
Configure the instances
```bash
# For bastion
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
sudo systemctl start chrony
sudo systemctl enable chrony
```
```bash
# For Nginx and webserver
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
sudo systemctl start chronyd
sudo systemctl enable chronyd

# Configure selinux policies
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db=1
sudo setsebool -P httpd_execmem=1
sudo setsebool -P httpd_use_nfs=1

# Install Amazon EFS utils for mounting EFS
git clone https://github.com/aws/efs-utils
cd efs-utils
sudo yum install -y make rpm-build
make rpm
sudo yum install -y  ./build/amazon-efs-utils*rpm

# Self-signed Certificate for Nginx
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/olalekan.key -out /etc/ssl/certs/olalekan.crt
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

# Self-signed Certificate for Apache
sudo yum install -y mod_ssl
openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/olalekan.key -x509 -days 365 -out /etc/pki/tls/certs/olalekan.crt
vim /etc/httpd/conf.d/ssl.conf
```
---
### Create AMIs from the instances
![ami](pbl15/ami.png)
---
### Create target groups
![target groups](pbl15/targetgroup.png)
---
### Create KMS key
![kms](pbl15/rdskey.png)
---
### Create RDS instance using the KMS key
![rds](pbl15/rds.png)
---
### Create certificate using the Amazon Certificate Manager
![certificate](pbl15/certicatemanager.png)

![record in hosted zone](pbl15/hostedzone.png)
