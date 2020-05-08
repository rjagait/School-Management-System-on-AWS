# School Management System on AWS
Deploying PHP based web app for school management with MySQL database, with an additional python image compressing code that gets triggered when an image is uploaded by student using the PHP web app. The compressed image then picked and updated on the yearbook.

## System Architecture
![architecture2](https://user-images.githubusercontent.com/55213734/81355770-7e36b080-909d-11ea-9ad8-635374e16754.png)

*Figure. For multiple EC2 instances of the web app deployed for Multi AZ database*

1.	School management system website developed in PHP (provided by the school)
1.	Load balancing with ELB (Elastic Load Balancer): Helps to spread across multiple Availability zones and EC2 auto-scaling groups for redundancy and decoupling of the services.
1.	Web server for PHP: Mostly Apache server is used.
1.	IAM User: Apart from updation access that the root user will have, all the regular users will be given access using an IAM user, whose role would be controlled by authentication and security and access provided.
1.	S3 buckets: S3 storage for data like images, that would be uploaded by the student. This can also be used for backup of the database items.
1.	Lambda function as a serverless application
1.	Managed Database with Amazon Aurora Db engine, with standby support for various available zones.
1.	Firewall with Security Groups for web servers

## Key components
1.	Load Balancing across clusters:
ELB (Elastic Load Balancing) helps manage the EC2 instances and can dynamically increase or decrease the connections as per demand and, helps distribute load across the available zones.

2.	Auto-triggered Python Image Compressor:
If we have 10 images uploaded at the same time, without lambda, all these images would be processed sequentially. Using lambda function instead would enable to process to process all the 10 images parallelly. Also, lambda gives us the functionality of easy implementation of event-based functionality. 
![Lambda flow](https://user-images.githubusercontent.com/55213734/81355979-0c129b80-909e-11ea-83d1-d96c2df66836.png)

  *Figure. Lambda Architecture*

3.	Database Configuration: 
Both Aurora and RDS Engine types can be used for MySQL. Comparing both below: 
o	Aurora provides higher throughput than MySQL
o	Both are compatible with MySQL database
o	Depending upon the usage, aurora will dynamically grow
o	Both support compute and the memory resource scaling
o	RDS has a limit of 5 replicas unlike Aurora. 


4.	Host Security:
The security groups in AWS can be used to control the traffic and access level for various users/functionalities across the process. These security groups can be assigned to one or more EC2 instances.
![network security](https://user-images.githubusercontent.com/55213734/81356009-251b4c80-909e-11ea-8588-156633f68dff.png)

*Figure. Security groups in web application*

5.	Network Management: 
Using Amazon VPC (Amazon Virtual Private Could) we can launch our applications in a private environment, thus isolating its network, thus providing a securer and more scalable architecture.

 
## Steps of Deployment
### Phase1: SQL Database setup
 ![phase1](https://user-images.githubusercontent.com/55213734/81356575-7ed04680-909f-11ea-8c2a-6ec7c25adb60.png)

*Figure. Part of architecture covered in phase1*

-	First, we’ll create an RDS Database using the RDS Services provided by AWS. Create a Standard Amazon Aurora instance with MySQL compatibility, with a regional database.
 
-	Ensuring security of the database we will create username and password that will only be added to the website deployed.
 
-	Choose the database instance size according to the need to the environment. Since it’s a small-scale demo app deployment for the case study, we can use default.

-	After creating Database instance, keep a note the security group’s info.
 
-	Add inbound rule to the security group to allow traffic from the aurora instance that we created.
 
### Phase2: Elastic Bean for the web server for php web application
 ![phase2](https://user-images.githubusercontent.com/55213734/81356615-9c9dab80-909f-11ea-8035-87a1e4c3f34e.PNG)

*Figure. Covered in 2nd phase*
Now we’ll add the Elastic Bean load balancer and deploy our PHP website.
-	Select the PHP Platform. Assuming, the php code provided is compatible with PHP 7.3 version
  
-	This created the EC2 instance, Instance security groups, Load balancer, Load balancer security groups, Auto Scaling group, Amazon S3 bucket (storage for the source code, logs, etc), Amazon Cloudwatch alarms (to monitor load on instances), AWS CloudFormation stack, Domain name (in the form of subdomain.region.elasticbeanstalk.com)

-	Wait for the instance to become healthy before doing the further configurations.
 
-	In addition to the security group that elastic bean creates, choose the security group to attach to the instances for the elastic bean environment.
 
-	Software configuration to add the RDS DB instance to elastic bean
 
-	When the health turns green, we will deploy the code and run the instance. Web app can be launched using the url shown in elastic bean -> Environments -> Instance details
 
### Phase3: Lambda Trigger for the python compressor app.
 ![phase3](https://user-images.githubusercontent.com/55213734/81356671-b8a14d00-909f-11ea-8200-3b9607b44cfc.PNG)

*Figure. Adding lambda to the architecture*

Image resizing serverless application. We will follow the below steps:
-	Setup IAM user. Create and assign a new role for the IAM user with permission to ‘AmazonS3FullAccess’

-	S3 bucket with 2 paths in it: one as the source image uploaded by student using the PHP app, and the other for destination image compressed by the python application. 
For the connectivity between the 2 applications, viz: PHP web service exposed to students to upload images and the python lambda service to compress the images to a smaller size, we can either stick to having 2 separate S3 containers, or we can move the connections of lambda python service to the same S3 bucket used in PHP web app. This decision would depend on the application requirements.

-	Creating a lambda function with python runtime.
 
-	Add in the trigger for the function, directing to the path of source images in the s3 container.
 
-	An event will be automatically generated in the source bucket.
 
-	Add the function code that compresses the image.
 
-	After lambda trigger deployment, the sequence of events for any image uploaded is expected to be as below:
 
 
## Benefits of moving to cloud
1.	Cost Reduction
As we start using cloud servers, there would be a cost reduction of the related maintenance of the servers. Since there is a range of available servers, the upgradation would be less costly, and simpler. Disposal and storage of old servers won’t be a concern for the user. These savings in total can go up to 25% or even more, depending on the setup.
1.	Capacity constraints
In case of a sudden increase in volume, like during result declaration or convocation when everyone wants to watch the live feed, there would a exceptional (short termed) demand for a higher bandwidth, which can be easily managed with cloud. As you can switch to a higher bandwidth and then switching back to the lower bandwidth, without worrying about buying the additional set of servers.
1.	Flexibility and learning support
Apart from the IT infrastructure, the school can also benefit from the additional services provided by cloud, like classes, talks, etc.
1.	Increased performance and fault tolerance
Backup and fault tolerance would be taken care of by cloud, hence less effort for the school.
