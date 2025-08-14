# aws-portfolio-project-4
Capstone Project

Task0:Inspecting your environment:
	Inspect the Example VPC
	Inspect the Subnets
	inspect the Security Groups
	Inspect the AMI
Open VPC and collect the Private and Public subnet Info

1)Subnet Info
	Public subnet1-10.0.0.0/24
	Public subnet2-10.0.1.0/24
	Private Subnet1-10.0.2.0/23
	Private Subnet2-10.0.4.0/23

2)Security Groups Info
	ALBSG-
	Bastion-SG-
	Example-DB-
	Inventory-App-

Task1: Create a MySql RDS database instance

Step1:Create Subnet Groups
	RDS-->Choose Subnet Groups
Subnet Group Details
	Name-Example-DB-subnet
	Description-Example-DB-subnet
	VPC-Select Example VPC
Add Subnet
	AZ-Select us-east-1a(private subnet 1) and us-east-1b1b(private subnet 2)
	Subnet-Select the Private subnet1(10.0.2.0/23) and Private subnet2(10.0.4.0/23)

Step2:Create Database
	Navigate RDS Service in aws and create db instance
	RDS-->Database-->Create Database

Create database:
	Choose a database creation method-Standard create
	Engine options-Select MySQL
	Templates-Dev/Test
	Availability and durability-Choose Multi-AZ DB instance
Settings
	DB instance identifier-Example
Credentials Settings:
	Master username-admin
	Master password-password
	Confirm password-password
Instance Configuration:
	DB instance class-Choose Burstable classes (includes t classes)-t3.micro
Storage:
	Storage type:General purpose
	Allocated Storage -20GB
Storage Autoscaling:
	Enable storage autoscaling

Connectivity
	Virtual private cloud (VPC)-Select Example VPC
	Subnet Group-Example-DB-subnet(automatically choosed)
	Public Access-No
	VPC Security Group-Example-DB

Database Authentication
	Database authentication options-Password authentication

Database options
	Initial database name-exampledb
	Backup-uncheck it
	Monitoring -Disable monitoring

Then click create database

Task2: Create Cloud9 Environment

Navigate cloud9 

Create clould environment

Step 1
Name environment
	Name-capstone project
Step 2
Configure settings
	Environment type-ensure Direct access

	Instance type-t2.micro (1 GiB RAM + 1 vCPU)

	Platform-Amazon Linux 2 (recommended)

Network settings (advanced)
	
	Network (VPC)-Select Example VPC

	Subnet-Select Public Subnet2
then click Next

Step 3
Review -Create Environment

Task3:Install a LAMP web server on Amazon Linux2 on cloud9 instance

Step 1: Prepare the LAMP server

	#sudo yum update -y
	#sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
	#sudo yum install -y httpd mariadb-server
	#sudo systemctl start httpd
	#sudo systemctl enable httpd
	#sudo systemctl is-enabled httpd

Step2-Download the project assets(copy from link from the capstone project)

//Download PHP Code:
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-ACACAD-20-EN/capstone-project/Example.zip

Then Come back to Cloud9 services and Unzip the php downloaded file by using 
	ls
	sudo unzip Example.zip 
	sudo cp Example/* /var/www/html/ 
	sudo rm Examble.zip

Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/

Check public IP of Cloud9 EC2 instance and paste into new tab(Now Webpage is not showing)

Solution:

Choose Instances and select your instance.(Cloud9 created Instance-Start with aws-cloud9)

On the Security tab, view the inbound rules. Add HTTP protocol with 0.0.0.0/0 then again refresh your webpage it will shows the webpage


Task4:Import the Databse into the database

	cd..

//Download Database:
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-ACACAD-20-EN/capstone-project/Countrydatadump.sql
	ll or ls

Again go to RDS dash board now your database instance is created(avilable)

and Copy the endpoint of DB

Go to Cloud9 service to access the machine

Database file is download successfully

Important:
	Go to Security Group and select Example-DB and add inbound rule for MYSQL/Aurora and source to cloud9 instance then only we can able to import the database

then come back cloud9 instance
   	mysql -u admin -p --host <rds-endpoint>

	DB instance identifier-Example
	Credentials Settings-admin
	Master password-password

1.copy and paste

 mysql -u admin -p --host <endpoint>

It asks password then give password(copy password from master password) then hit enter

If correct means it will show like this MySQL [(none)]>  
	then type -show databases; then it will shows the tables in database
	MySQL [(none)]>  use exampledb;
	MySQL [exampledb]> show tables;
	MySQL [exampledb]> exit;
	MySQL [exampledb]> exit;
	Bye
	voclabs:~/environment $ 

Import the file:

	mysql -u admin -p exampledb --host <rds-endpoint> < Countrydatadump.sql

It asks password then give password(copy password from master password) then hit enter

Verify once again database is attached or not by using following command


	mysql -u admin -p --host <endpoint>

	MySQL [(none)]>  use exampledb;
	MySQL [exampledb]> show tables;
	MySQL [exampledb]> select*from countrydata_final;
	MySQL [exampledb]> exit;

Task5:Configure Parameter values in AWS systems manager

Step1:
Navigate to AWS systems manager and create parameter for following values

	/example/endpoint   <rds-endpoint>
	/example/username  admin
	/example/password   password
	/example/database   exampledb

Step2:Modify IAM role to cloud9 instance

Go to cloud9 EC2 instance and attach IAM role-Inventory-App-Role
then refresh the web page again
Now you can access the database successfully.

Task6:Create AMI for Autoscaling(Cloud9 instance)

In this step we take an AMI on cloud9 instance

select cloud9 instance-->actions-->images and templates-->create image
	
	Image Name-CapstoneProjectAMI
	Description-AMI for CapstoneProject
Then create Image.
It will takes few minutes.

Task7-Create Load Balancer
 Step1:
Create Load Balancer

Go to EC2 console and select Load Balancer in new tab
	Select Create Load Balancer
	Select Application Load Balancer
		Load balancer name-CapstoneProject-LB
Network mapping
	VPC-Select Example VPC
	Mappings-Select both us-east-1a below subnet choose Public subnet1  & us-east-1b below subnet choose Public subnet2

Security groups
	Security groups-Select ALBSG security group

Listeners and routing
	Default action-Create Load Balancer

Create Target Group:
Step 1
Specify group details:
	Choose a target type-instance
	Target group name-CapstoneProject-TG
	VPC-Ensure Example VPC is selected then click next
Step 2
Register targets:
	2 avilable Target is there so click create target Group(Don't select instances)

Come back to Load balance and refresh this Listener and routing,now we can see created target group name and select it.

	Select CapstoneProject-TG
Then Click create Load Balancer

Copy the DNS Name

Task8-Create AutoScaling

EC2 management console under Auto Scaling choose Auto Scaling Groups in new tab

Create Auto Scaling group
Step 1
Choose launch template or configuration
Name
	Auto Scaling group name-NmindAcademy-ASG
Launch template
	Launch template-Choose Example-LT then modify the template
	Change the AMI ID just now we created as CpstoneProjectAMI

Select Example-LT go to details-->Actions-->Modify Templates

Scroll down and on Launch Templates Contents choose our CapstoneProjectAMI ID then create it.

Ensure the CapstoneProjectAMI ID is changed in out template then click next

Step 2
Choose instance launch options
Network
	VPC-Select Example VPC
Availability Zones and subnets-Select Private subnet1 & Private subnet2 then click next

Step 3 (optional)
Configure advanced options
	Load balancing - optional-Select Attach to an existing load balancer

Attach to an existing load balancer
	Existing load balancer target groups-Select CapstoneProject-LB
	Health checks - Select ELB(check mark) then click next

Step 4 (optional)
Configure group size and scaling policies

	Group size -Increase the group size like below
Desired capacity
2
Minimum capacity
2
Maximum capacity
2

Then click Next

Step 5 (optional)
Add notifications

Then click Next

Step 6 (optional)
Add tags
	Add name tag and value as Nminds-CapstoneProject and click next

Step 7
Review  then click Create Auto Scaling group

Task9:Check the ouput by using load balancer DNS name

Go to Load balancer and copy that DNS name and paste it into new tab.

And check website and database are connected or not.



Submit Your Work.










