# AWS--Migration

![Migration](https://user-images.githubusercontent.com/19390842/163001524-19e2a1f6-0aaa-4d45-8cf6-083ae8798708.PNG)

In this project, you will be responsible for migrating a work load running in an Enterprise Data Center to AWS. Your mission will be to migrate the application and its database using Amazon EC2 and RDS services. In the solution architecture on AWS, it will be necessary to implement a VPC along with its Subnets, an EC2 instance to store the Application, and an RDS instance that will store the Database. For your application to be accessible via the Internet, attached to your VPC, you will need to implement Internet Gateway. 

Hands-on Project Solution - Part 1

Implementation

# VPC Provisioning

- Create a VPC: VPC with a Single Public Subnet

- CIDR Block: 10.0.0.0/16
PS: It should NOT overlap with on-premises or other cloud providers

- VPC Name: vpc-production-1

- Public subnet details:

vpc-production-1-pu-1 (Public)
CIDR Block: 10.0.0.0/24

- Private subnet details:

vpc-production-1-pv-1 (Private)
CIDR Block: 10.0.1.0/24

vpc-production-1-pv-2 (Private)
CIDR Block: 10.0.2.0/24

# EC2 Provisioning

- Create a new EC2 instance

- Amazon Machine Image: Ubuntu 18.04
- Instance type: t2.micro
- Instance details:
- Network: vpc-production-1
- Subnet: vpc-production-1-pu-1 (Public)
- Assign public ip: Enable
- Tags:
- hostname: awsuse1app01
- environment: bootcamp
- Configure security group:
- Port: 22 - Source: 0.0.0.0/0
- Port: 8080 - Source: 0.0.0.0/0

# RDS Provisioning

- Create a RDS instance

- MySQL
- MySQL 5.7.30
- Templates: free-tier
- Settings:
- DB Instance: awsuse1db01
- Master username: admin
- Master password: admin123456
- DB instance class: db.t2.micro
- Storage: 20 GB
- Connectivity:
- VPC: vpc-production-1
- Public access: No
- Create new VPC security group
- New VPC security group name: sec-group-db-01
- Availability Zone: us-east-1a


Hands-on Project Solution - Part 2

# Pre-requisites steps to run in the EC2 instance (Application Server)

sudo apt-get update
sudo apt-get install python3-dev -y
sudo apt-get install libmysqlclient-dev -y
sudo apt-get install unzip -y
sudo apt-get install libpq-dev python-dev libxml2-dev libxslt1-dev  libldap2-dev -y
sudo apt-get install libsasl2-dev libffi-dev -y

curl -O https://bootstrap.pypa.io/get-pip.py ; python3 get-pip.py --user

export PATH=$PATH:/home/ubuntu/.local/bin/

pip3 install flask
pip3 install wtforms
pip3 install flask_mysqldb
pip3 install passlib

sudo apt install awscli -y
sudo apt-get install mysql-client -y



Hands-on Project Solution - Part 3

# Steps to run during the "dry-run & cutover":

- Open port RDS security group:

- Type: MySQL/Aurora
- Source: 0.0.0.0/0
(only for bootcamp purporses - in production env, you should open the ports to the application server IPs only)

- Connect to the EC2 instance

$ ssh ubuntu@<PUBLIC_IP> -i <ssh_private_key>

- Download the dump file and app file

wget https://tcb-bootcamps.s3.amazonaws.com/bootcamp-aws/en/wikiapp-en.zip
wget https://tcb-bootcamps.s3.amazonaws.com/bootcamp-aws/en/dump-en.sql

- Connect on MySQL running on AWS RDS

mysql -h <RDS_ENDPOINT> -P 3306 -u admin -p

- Create the wiki DB and import data

create database wikidb;
use wikidb;
source dump-en.sql;

- Create the user wiki in the wikidb

CREATE USER wiki@'%' IDENTIFIED BY 'wiki123456';

GRANT ALL PRIVILEGES ON wikidb.* TO wiki@'%';

FLUSH PRIVILEGES;

- Unzip the app files

unzip wikiapp-en.zip
cd wikiapp

- Edit the wiki.py file and change the MYSQL_HOST and point to the RDS endpoint.

vi wiki.py

- Bring up the application

python3 wiki.py

Steps to validate the migration:

- Open the AWS console, copy the public IP and test the application using:

<EC2_PUBLIC_IP>:8080


- Login to the application (user:admin / password: admin)

- Create a new article
