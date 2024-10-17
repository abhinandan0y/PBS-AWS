# AWS Example

#### From creating an account to deploying and running your application. 
#### Step-by-step guide to help you get started:

#### Step 1: Create an AWS Account
```
Sign Up: Go to the AWS website and click on "Create an Account."
Fill in Details: Provide your email address, password, and other required information.
Payment Information: Even though you have a free account, you need to provide payment information. AWS will not charge you unless you exceed the free tier limits.
Verify Your Identity: You may need to verify your phone number and complete a CAPTCHA.
Select Support Plan: Choose the Basic (Free) support plan.
Confirm Account: Complete the sign-up process and confirm your account via email.
```
  <img src="https://raw.githubusercontent.com/abhinandan0y/PBS-AWS/refs/heads/AWS-Example/aws-ref/Screenshot%20from%202024-10-18%2000-26-48.png?token=GHSAT0AAAAAACXY7ZKHRY4D7EO2VLUREJTGZYROYIA" style="width: 100%;" alt="bioinformatics_lab.png">
  
#### Step 2: Set Up Your AWS Environment
```
Log In: Go to the AWS Management Console and log in with your credentials.
IAM Setup: Create an IAM user with appropriate permissions for better security. Go to the IAM service and create a new user with programmatic access.
```

#### Step 3: Choose a Service
```
For running a program, you can choose from various AWS services. Let’s use Amazon EC2 (Elastic Compute Cloud) for this example.
```

#### Step 4: Launch an EC2 Instance
```
Navigate to EC2: In the AWS Management Console, go to the EC2 Dashboard.
Launch Instance: Click on "Launch Instance."
*** Choose an AMI ***: Select an Amazon Machine Image (AMI). For example, choose "Amazon Linux 2 AMI."
Choose Instance Type: Select a free-tier eligible instance type, such as t2.micro.
Configure Instance: Review the default settings and click "Next: Add Storage."
Add Storage: Use the default storage settings and click "Next: Add Tags."
Add Tags: Optionally, add tags to your instance for better organization.
Configure Security Group: Create a new security group or use an existing one. Make sure to allow SSH access (port 22) for remote access.
Review and Launch: Review your settings and click "Launch."
Create Key Pair: Create a new key pair or use an existing one. Download the key pair file (.pem).
```
#### Step 5: Connect to Your EC2 Instance
```
Get Public DNS: In the EC2 Dashboard, find your instance and note its Public DNS.
SSH into Instance: Use a terminal or SSH client to connect to your instance. For example:

ssh -i /path/to/your-key-pair.pem ec2-user@your-instance-public-dns
```
#### Step 6: Run Your Program
```
Upload Your Program: Use scp or another method to upload your program files to the EC2 instance.

scp -i /path/to/your-key-pair.pem /path/to/your/program.py ec2-user@your-instance-public-dns:/home/ec2-user/
Run Your Program: SSH into the instance and run your program.

python3 program.py
```
#### Step 7: Monitor and Manage Costs
```
Free Tier Limits: Ensure you stay within the free tier limits. For EC2, the free tier includes 750 hours of t2.micro or t3.micro instance usage per month.
Billing Dashboard: Go to the AWS Billing Dashboard to monitor your usage and costs.
Set Up Alerts: Set up billing alerts to notify you if your usage exceeds the free tier limits.
Step 8: Clean Up Resources
Terminate Instance: When you’re done, terminate your EC2 instance to avoid unnecessary charges.
Delete Resources: Delete any other resources you created, such as security groups or key pairs.
Pricing Information
Free Tier: AWS offers a free tier that includes 750 hours of t2.micro or t3.micro instance usage per month for the first 12 months.
Beyond Free Tier: If you exceed the free tier limits, you will be charged according to the standard pricing. You can find detailed pricing information on the AWS Pricing page.
By following these steps, you can successfully run a program on AWS using a free account. Make sure to monitor your usage to stay within the free tier limits and avoid unexpected charges.
```
