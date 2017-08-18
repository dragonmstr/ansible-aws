# ansible-aws
Ansible for AWS

# Incomplete will be finished by 17 Aug
- Create CleanUP Folder
- Fix Ubuntu 16.04 deploy or Packing AMI with Packer
- Organise README.md
-

# Task below:

Task:
Write an Ansible Role that includes a number of tasks:
- Power up an RDS instance and set the security group so that only the EC2 instance (below) can connect to it on port 3306 (via internal IP addresses)
- Power up an EC2 instance, running Ubuntu 16.04 and the latest version of the software package mysql-client

Instructions should be provided on how to run the role, including how we can pass in the following parameters:
- instance sizes
- AWS region
- AWS creds
- DB creds

We will run on one of our test AWS environments. The goal here is to produce a reusable role.

## Prerequisites:

1. Ansible Workstation Setup
2. AWS Authentication with Ansible
3. Building the AWS Network
4. Building the Bastion
5. Building RDS
6. Building the MySQL-Client

- This playbook deploys the whole AWS Network, Bastion, MySQL-Client, and RDS Instances.  MySQL-Client Configuration is currently manual
- Must have credentials exported for AWS IAM
- RDS may take up to 30 minutes to deploy the instance

# Ansible Workstation Setup

## Control Machine:
The control machine acts as the Ansible server where all the playbooks and configuration files are located. It can be configured on common Linux Distribution, OS X or BSDs. For this example, we have configured Ubuntu OS on the workstation.

#### To setup the ansible workstation the following commands are used

```
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

#### Configuring the workstation- The Ansible AWS modules uses the Python Boto library internally.
The following commands are used to set up the boto run

```
$ sudo easy_install pip
$ sudo pip install boto
```

# AWS Authentication with Ansible

After installation, create a file named boto and provide the necessary credentials.

Create file ~/.boto
```
     [Credentials]
     aws_access_key_id = (access key)
     aws_secret_access_key = (secret access key)
```

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely.
```
$ ansible-playbook -i hosts keypair.yml
```

To run Ansible on your workstation, you need to set environment variables by specifying your Secret Key and Access Key.

```
$ export AWS_ACCESS_KEY_ID= 'YOUR_AWS_API_KEY'
$ export AWS_SECRET_ACCESS_KEY=  'YOUR_AWS_API_SECRET_KEY'
```

# Ansible Imagination Dev Site
To help make the roles reusable and easily updated, the variables were placed in the main site.yml file for configuring all of the aspects from the network, bastion, and RDS.

*Site.yml*
```
---
- name: Deploy RDS Infratructure
  hosts: localhost
  connection: local
  gather_facts: false
  vars:

# Global AWS Variables
    region: eu-west-1
    aws_network_name: dev

# VPC Variables
    vpc_cidr: 10.4.0.0/16
    public_subnet_cidr: 10.4.0.0/24
    public_subnet_az: eu-west-1a
    private_subnet_1_cidr: eu.4.1.0/24
    private_subnet_1_az: eu-west-1b
    private_subnet_2_cidr: 10.4.2.0/24
    private_subnet_2_az: eu-west-1c

# Bastion Variables
    ec2_id: "ami-785db401"
    key_pair: "imagination-key"

# RDS Variables
    rds_user: root
    rds_pass: Default_Pass123!
    rds_instance_type: db.m1.small
    rds_size_gb: 15
    rds_parameter_engine: mysql5.6
    rds_instance_engine: 5.6.34
    rds_parameters:
      - { param: 'binlog_format', value: 'ROW' }
      - { param: 'general_log', value: '1' }

  roles:
#  - common
  - aws-network
  - aws-bastion
#  - aws-rds
```

# Building the AWS Network

– Build VPC
– Build Subnets
– 1 Public subnet to access the environment
– 2 Private subnets for internal traffic. Two because the RDS Subnet group requires two for redundancy.
– Internet Gateway for the VPC
– Routing Table to allow the three subnets to talk to one another and then send any non subnet traffic to the Internet Gateway
– Security groups to allow specific traffic into specific instances

## Building the MySQL-Client
```

```

## Connecting to a Bastion Host
```
$ ssh -i ~/.ssh/imagination-key.pem ubuntu@ec2-54-171-69-84.eu-west-1.compute.amazonaws.com
```
## Connecting to a DB Instance Running the MySQL Database Engine
```
$ ssh 10.4.1.175 -F ssh.cfg
$ mysql -h myinstance.123456789012.eu-west-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
```

## CleanUP


## Conclusion:

Ansible integrates with AWS to build the network and instances. The Ansible AWS modules make it easy to manage and configure Amazon’s EC2 instances.

## Issues:
- If you're using Ansible >2.2.0, you can set the ansible_python_interpreter configuration option to /usr/bin/python3:
```
ansible my_ubuntu_host -m ping -e 'ansible_python_interpreter=/usr/bin/python3'
```
https://docs.ansible.com/ansible/latest/python_3_support.html

- decided to upgrade to the latest version of ansible (2.3.0.0). Then I created a...

```
group_vars/all

..file and added...

ansible_python_interpreter: /usr/bin/python3
```

# Sources:
- https://blog.scottlowe.org/2015/12/11/using-ssh-multiplexing/
- https://www.cyberciti.biz/faq/linux-unix-osx-bsd-ssh-multiplexing-to-speed-up-ssh-connections/
- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html
