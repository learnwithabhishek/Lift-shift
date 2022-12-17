# Lift-shift
About the project:
•	Multi-tier web application stack (VPROFILE)
•	Host & run-on AWS cloud for production 
•	Lift & shift strategy

SCENARIO:
Application services running on physical/virtual machines
Work load is in your datacentre. 
PROBLEM:
•	Complex management. 
•	Scale up/down complexity.
•	Upfront capital expenditure & regular operations expenditure
•	Manual process
•	Difficult to automate
•	Time consuming

Solution:
Cloud computing setup
AWS Services 
•	EC2 instances: VM for TOMCAT, RabbitMQ, Memcached, MySQL
•	ELB (Load Balancer)
•	Autoscaling: Automating VM scaling
•	S3/EFS Storage: Shared storage
•	ROUTE 53: For private DNS Service
•	IAM, ACM, EBS also used

![image](https://user-images.githubusercontent.com/110404399/208224307-09114eaf-ee03-4799-b41d-7436f00f7216.png)
