Las Vegas Basketry AWS CloudFormation Project

Project Summary

This project demonstrates how I designed, deployed, tested, troubleshot, and cleaned up a multi-tier AWS application using AWS CloudFormation.

The goal was to recreate a manually built AWS environment as Infrastructure as Code (IaC). The application allows a customer to submit a gift basket order through a web form. The request is routed through an internet-facing Application Load Balancer to a private EC2 application server running Apache and PHP. The PHP application connects to an Amazon RDS for MySQL database and stores the order in an orders table.

Architecture

Customer
   |
   v
Application Load Balancer
   |
   v
Target Group
   |
   v
Private EC2 Instance
Apache + PHP
   |
   v
Amazon RDS MySQL
   |
   v
orders table

The application server and database are private. The Application Load Balancer is the public entry point.

Project Objectives

Build a secure AWS network using public and private subnets

Deploy an application server without assigning it a public IP address

Use Systems Manager Session Manager instead of direct SSH access

Route customer traffic through an Application Load Balancer

Connect a PHP application to an RDS MySQL database

Rebuild the environment using AWS CloudFormation

Validate the application end-to-end

Troubleshoot deployment and connectivity issues

Remove billable AWS resources after testing

AWS Services Used

AWS CloudFormation

Amazon VPC

Public and Private Subnets

Internet Gateway

NAT Gateway

Elastic IP

Route Tables

Security Groups

Amazon EC2

AWS Identity and Access Management (IAM)

AWS Systems Manager Session Manager

Application Load Balancer

Target Groups

Amazon RDS for MySQL

Network Design

The CloudFormation template creates a VPC with the CIDR block:

10.50.0.0/16

The environment contains:

Two public subnets

Two private subnets

An Internet Gateway

Public and private route tables

A NAT Gateway for outbound internet access from private resources

The Application Load Balancer is deployed in the public subnets.

The EC2 application server and RDS database are deployed in private subnets.

Security Design

The project uses security groups to restrict traffic between application tiers.

Application Load Balancer Security Group

Allows inbound web traffic on:

TCP 80

TCP 443

EC2 Application Security Group

Allows inbound HTTP traffic on port 80 only from the Application Load Balancer security group.

RDS Security Group

Allows inbound MySQL traffic on port 3306 only from the EC2 application security group.

EC2 Management

The EC2 instance uses an IAM role with:

AmazonSSMManagedInstanceCore

This allows the server to be managed with AWS Systems Manager Session Manager without exposing SSH access to the internet.

Application Server

The EC2 application server runs Amazon Linux and hosts the Basketry application.

CloudFormation launches the instance in a private subnet and attaches:

The application security group

The Systems Manager IAM instance profile

EC2 UserData installs:

Apache HTTP Server

PHP

PHP MySQL support

The application files are stored under:

/var/www/html

The database configuration file is stored outside the public web root:

/etc/lasvegas-basketry-db.php

Database

The project uses Amazon RDS for MySQL.

Application database:

lasvegas_basketry_orders

Primary table:

orders

The orders table stores:

Order ID

Customer name

Phone

Email

Delivery address

Occasion

Basket size

Recipient name

Gift message

Special requests

Delivery date

Delivery time

Order status

Created timestamp

CloudFormation Lab Progression

Lab 1 — VPC

Created the project VPC.

Lab 2 — Subnets

Created two public and two private subnets across Availability Zones.

Lab 3 — Routing

Created:

Internet Gateway

Public route table

Private route table

Subnet route table associations

Lab 4 — Security Groups

Created the Application Load Balancer and application server security groups.

Lab 5 — NAT Gateway

Created:

Elastic IP

NAT Gateway

Private default route

This provided outbound internet access for resources in the private subnets.

Lab 6 — IAM and Systems Manager

Created:

EC2 IAM role

Instance profile

Attached AmazonSSMManagedInstanceCore to support Session Manager.

Lab 7 — Private EC2 Application Server

Created the EC2 application server with CloudFormation.

Automated installation of Apache and PHP and verified the server using Session Manager.

Lab 8 — Target Group

Created an Application Load Balancer target group and registered the EC2 instance.

Verified the target reached a healthy state.

Lab 9 — Application Load Balancer

Created:

Internet-facing Application Load Balancer

HTTP listener

Forwarding rule to the target group

Verified the application was reachable through the ALB DNS name.

Lab 10 — RDS Networking

Created:

RDS security group

RDS DB subnet group

Configured MySQL port 3306 to accept traffic only from the EC2 application security group.

Lab 11 — RDS MySQL

Created the RDS MySQL database through CloudFormation.

The database is:

Private

Encrypted

Associated with the private DB subnet group

Protected by the RDS security group

Lab 12 — EC2-to-RDS Connectivity

Connected from the EC2 instance to RDS.

Created:

lasvegas_basketry_orders

and the:

orders

table.

Lab 13 — PHP Order Application

Connected the PHP application to RDS.

Created:

Customer order form

PHP database connection

Prepared SQL insert

Order confirmation page

Submitted a test order through the Application Load Balancer and verified the new record directly in the MySQL orders table.

End-to-End Test

The final application flow was validated successfully:

Customer
   |
   v
Application Load Balancer
   |
   v
Target Group
   |
   v
Private EC2
   |
   v
Apache
   |
   v
PHP
   |
   v
Amazon RDS MySQL
   |
   v
orders table

A test order submitted through the browser was successfully stored in MySQL.

Troubleshooting Experience

Several issues were identified and resolved during the project.

YAML Formatting

Resolved CloudFormation template errors caused by incorrect YAML indentation and resource placement.

Invalid CloudFormation Property

Corrected the EC2 IamInstanceProfile property after CloudFormation reported an invalid object type.

IAM Capabilities

Handled CloudFormation IAM capability acknowledgements when creating IAM resources.

RDS Account Limit

Troubleshot an RDS ServiceLimitExceeded error caused by the account's free-plan database instance limit.

Target Group Health

Verified Apache availability and target group health before connecting the load balancer.

PHP-to-RDS Connection

Troubleshot an HTTP 500 error caused by an incorrectly formatted PHP database configuration file.

Used commands such as:

php -l
curl
systemctl is-active httpd
mysql

to isolate and resolve application and connectivity problems.

Database Verification

Verified application inserts directly in MySQL with SQL queries against the orders table.

Cost Management

After completing the project, the CloudFormation stack was deleted to remove billable resources.

Cleanup included resources such as:

NAT Gateway

Elastic IP

Application Load Balancer

Target Group

EC2 instance

RDS database

Security groups

Subnets

Route tables

Internet Gateway

VPC

A post-cleanup review was also performed to check for older manually created resources.

Skills Demonstrated

This project demonstrates hands-on experience with:

AWS networking

Infrastructure as Code

AWS CloudFormation

VPC architecture

Public and private subnet design

Linux administration

Apache configuration

PHP application hosting

IAM roles and instance profiles

Systems Manager Session Manager

Load balancing

Target health checks

Amazon RDS

MySQL

Security groups

Application-to-database connectivity

Cloud troubleshooting

Resource cleanup

AWS cost awareness

Interview Summary

A concise way to describe this project in an interview:

I built a multi-tier gift basket ordering application in AWS and then recreated the environment with CloudFormation. The architecture used an internet-facing Application Load Balancer, a private EC2 instance running Apache and PHP, and a private RDS MySQL database. I configured VPC networking, security groups, NAT access, IAM roles, Systems Manager, target groups, and database connectivity. I also troubleshot CloudFormation, PHP, RDS, and networking issues and validated the full customer-to-database workflow before cleaning up the environment.

Future Enhancements

Potential next steps include:

HTTPS with AWS Certificate Manager

Route 53 DNS

AWS Secrets Manager for database credentials

CloudWatch metrics and alarms

Auto Scaling

CI/CD pipeline

S3 for product images

AWS WAF

CloudFront

Development, test, and production environments

Automated database initialization

Improved frontend styling

Application logging and monitoring

Repository Structure

lasvegas-basketry-cloudformation/
|
|-- README.md
|-- basketry-vpc.yaml

Notes

This project was created as a hands-on AWS learning and portfolio project. Production environments should use stronger secret-management, monitoring, backup, availability, and deployment practices.