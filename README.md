# Deploying-a-3-tier-web-application-in-AWS

This project involves the deployment of an application where users can only access servers through a load balancer. I made use of an application load balancer on a public subnet to balance traffic across my servers. Servers and databases (MYSQL) were placed in private subnets across multiple availability zones (AZ), thereby providing a high level of reliability, redundancy, scalability, security, and availability.

# STEP1: VPC

Create a VPC with 10.0.0.0/16
Edit created vpc and enable DNS hostname
Create an internet gateway and attach to the created vpc

Create 9 subnet (3 public for web tier, 3 private for app, 3 private for data tier) 10.0.0.0/24
10.0.1.0/24, 10.0.2.0/24 etc
Edit public/web subnet and enable auto assign ipaddress 

Create 3 route tables for each tier
Create nat gateway in the public subnet and allocate an elastic ip
Edit web/public route and add internate gateway
Attach the 3 public subnets to the public route through subnet association

create nat gateway and attach 3 created app subnet to app route
create nat gateway and attach 3 created data subnet to data route

# STEP2: SERVER

Create an EC2 instance (linux2 AMI) for a baston host with created vpc & public subnet ( baston host; a server that provides secure access to other server but not directly accessible from the internet or other networks)
Create an EC2 instance(linux2 AMI) for app server with created vpc & private app subnet. Create new security group that uses the created baston host sg as source info and a private .pem keypair
Create another EC2 instance(Linux2 AMI) for app server 2 with created vpc & 2ns private app subnet. Choose security group of EC2 app server1

ssh into baston host
Create a new file “keypair.pem” the copy the content of the private key into the file
Make file executable: chmod 400 keypair.pem
Ssh into app server: ssh -i keypair.pem ec2-user@ip appserver    

From AWS Documentation: Install LAMP (Linux, Apache, MariaDB & PHP) on Linux 2
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html

Confirm that apache webserver is working: curl http://localhost
Use the documentation to complete configuration.

Perform same installation in app server 2

Create an Application Load Balancer with public subnets. Create a security group. Create an instance target group  running app server only
Attach created security group of Load balancer to app server1 inbound sg

Test to see Load Balancer is working : 
cd /var/www/html	(both server)
		Echo “my server1 is working” > index.html
		Echo “my server 2 is working” > index.html
		Browse DNS of LB. working? Refresh again

![image](https://github.com/Edosaig/Deploying-a-3-tier-web-application-in-AWS/assets/107155943/d3e4dc82-5450-445c-bbf4-1f093736f293)

![image](https://github.com/Edosaig/Deploying-a-3-tier-web-application-in-AWS/assets/107155943/69b78ed7-5426-4a79-bd5c-1adc09487ca4)

![image](https://github.com/Edosaig/Deploying-a-3-tier-web-application-in-AWS/assets/107155943/ecb51514-26ae-4b0b-b738-f72e5741ce9a)

![image](https://github.com/Edosaig/Deploying-a-3-tier-web-application-in-AWS/assets/107155943/c7aacd27-4d87-4c84-b9ad-fba46876a31f)

# STEP3: DATABASE

Create db subnet group in RDS, choose project VPC with 3 AZ and choose the subnet for the private db subnet.
Create MYSQL database, Dev/Test as template. Multi-AZ DB. DB Class: bustable
Storage: general purpose ssd. Disable storage auto scaling. Select project vpc. 
Public access: no
Create security group

Modify the inbound rule of database security group: CustomTCP, port 3306 and app server sg
Cd /var/www/html
Cd phpMyAdmin
Rename php config file: mv config.sample.inc.php config.inc.php
Modify the configuration file: Vi config.inc.php and replace localhost with database endpoint
Enable stickiness from attribute in target group
Browse ALB DNS/phpMyAdmin

![image](https://github.com/Edosaig/Deploying-a-3-tier-web-application-in-AWS/assets/107155943/980f14d0-1e90-4abd-834c-9a4e79782eb9)
