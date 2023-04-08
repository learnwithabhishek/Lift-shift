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

![image](https://user-images.githubusercontent.com/110404399/208224648-545f14cc-9892-4fe1-8bbd-1e5b7ba48b71.png)

# Steps

Create key pair of instances

Create security groups – ALB_SG(Allow https from all ipv4 and ipv6) , vrofile_app_SG (allow 8080 from ALB_SG), Backend_SG(Allow all traffic from vprofile_app_SG , allow all traffic from own sg)
![image](https://user-images.githubusercontent.com/110404399/230716087-c00af167-3ce8-47a7-aacb-2d6aa2a43963.png)
![image](https://user-images.githubusercontent.com/110404399/230716098-5d10f54a-57b0-4b9a-a44d-c40fc821fd25.png)
![image](https://user-images.githubusercontent.com/110404399/230716115-dbd979b5-8a67-49a3-8900-871cb800b050.png)

Create db01, mc01, rmq01 instance with centos7 ami and configure it with "vprofile-project/userdata/ mysql, memchache, rabbitmq files". All of these instances will have backend_SG attached.

Create private hosted zone, vprofile.in, in Route53. Create A records: db01.vprofile.in to private ip of db01, mc01.vprofile.in to private ip of mc01, rmq01.vprofile.in to private ip of rmq01 :
![image](https://user-images.githubusercontent.com/110404399/230716363-11102224-720a-459d-9627-3112c3361242.png)

Update vprofile-project/src/main/resources/application.properties file :
![image](https://user-images.githubusercontent.com/110404399/230716427-d9ff6c16-36bd-4478-88e1-4a2ea8cf8941.png)

Create vprofile-app instance using ubuntu 20.04, install tomcat9, from vprofile-project/userdata/tomcat_ubuntu file

Run MVN clean install in the directory where pom file located to get the artifact

![image](https://user-images.githubusercontent.com/110404399/230716521-fcd97252-a689-4b27-883e-c55c4012269b.png)


Upload the artifact(vprofile-project/target/vprofile-v2.war) to s3 bucket
![image](https://user-images.githubusercontent.com/110404399/230716604-d7ec472e-46d8-4b47-b0d8-62f8fc0f8e40.png)


Create a role with s3 full access and attach the role to vprofile-app instance. Install awscli.
![image](https://user-images.githubusercontent.com/110404399/230716687-557d870a-e60a-4384-a881-58f9f12db740.png)

In vprofile-app instance remove ROOT from /var/lib/tomcat9/webapps/

Stop tomcat9: (sudo systemctl stop tomcat9)

Copy the artifact vprofile-v2.war from s3 bucket to vprofile-app instance at /var/lib/tomcat9/webapps/ROOT.war location

Now start tomcat9: (sudo systemctl start tomcat9)

![image](https://user-images.githubusercontent.com/110404399/230716935-e7f11ac8-85ed-44af-93df-72bd589daf04.png)

Create AMI image from vprofile-app instance

Then create a launch template using the above AMI 

Create a target group which do health check on port 8080, include vprofile-app instance into it

Then create application load balancer using above target group. Add listeners at port 443(https) 
![image](https://user-images.githubusercontent.com/110404399/230716987-d8a977f4-9543-41d0-bc97-f1dbdc044d01.png)

Add SSL/TLS certificate for https traffic

![image](https://user-images.githubusercontent.com/110404399/230717020-58d29a50-997a-4a26-9709-50a62d249e7b.png)

After load balancer created, go to dns registrar, (ex. Godaddy) create a CNAME DNS record with name vprofile and value will be load balancer dns name.
![image](https://user-images.githubusercontent.com/110404399/230717052-6038bbae-5eee-43a8-93d6-ebdb8662015c.png)

After all health checks completion you can access your vprofile application at "vprofile.yourdomain"

![image](https://user-images.githubusercontent.com/110404399/230717110-dff43256-b3ac-4f9c-8540-db3414c723b3.png)

login with username: admin_vp and password: admin_vp
![image](https://user-images.githubusercontent.com/110404399/230717181-96f39141-6fb3-49a3-88ad-3f1338104c20.png)

Click on Rabbitmq, if backend rmq01 instance running and configured properly then will receive msg:
![image](https://user-images.githubusercontent.com/110404399/230717267-d3f69c22-abf7-4506-b51d-fea15d28a123.png)

Click allusers then click on any user, if you get response then db01 instance working and connected properly:
![image](https://user-images.githubusercontent.com/110404399/230717377-677a69fd-6f0b-4801-876e-3f831a3aa12c.png)

Click back button, and select same user. Now you get cached data from memcache:
![image](https://user-images.githubusercontent.com/110404399/230717421-51c1d62d-81a1-4643-8208-d4435ccaa0ad.png)


