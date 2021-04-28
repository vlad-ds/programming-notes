# AWS Regions

AWS has regions all around the world. `us-east-1`, `eu-west-1` etc.

A region is a cluster of data centers. Most AWS services are region-scoped (if you use the same service in another region, you will not have your data replicated or synched). 

Availability Zones (AZ). Each region has 2, 3 or 6 AZs. Indicated by letters. For example Sydney has `ap-southeast-2a`, `ap-southeast-2b`, `ap-souteast-2c`. Each AZ is one or more discrete data centers with redundant power, networking and connectivity. AZs are separated from each other, and isolated from disasters. They are however all connected with high bandwidth, ultra-low latency networking.

Check https://aws.amazon.com/about-aws/global-infrastructure/?p=ngi&loc=0 to check Regions and AZs and see where services are available. 


# IAM

Identity and Access Management. All AWS security is here:

* Users. Physical persons. 
* Groups. Functions (admins, devops). Teams (engineering, design). Apply permissions to groups and their users will inherit them.
* Roles. Internal usage within AWS resources. We give roles to machines. 

Root account should never be used and shared. Users must be created with proper permissions. IAM has policies written in JSON which define what Users, Groups and Roles can or cannot do. 

IAM has a **global** view. 

It is possible to enable MFA (Multi Factor Authentication). 

**Least privilege principles.** Give users the minimal permissions they need to perform their job. 

**IAM Federation**. Big enterprises usually integrate their own repository of users with IAM. So people can login into AWS using their company credentials. This uses the **SAML standard** (Active Directory). 

Rules:

* One IAM User per physical person
* One IAM Role per application
* IAM credentials must never be shared
* Never write IAM credentials in code
* Never commit IAM credentials
* Never use ROOT account except for initial setup

**IAM hands-on**. Created admin user, added to group, created IAM sign-in link.

# EC2

* Renting virtual machines in the cloud (EC2)

* Storing data on virtual drives (EBS)

* Distributing load across machines (ELB)

* Scaling the services using an auto-scaling group

  

1. 
   Go to the EC2 dashboard and launch an instance. 
2. You need to chose an AMI (Amazon Machine image). This is the software that will be launched on a server. The Amazon Linux 2 AMI is recommended. 
3. Choose an instance type. How powerful the machine is. 
4. Configure parameters. 
5. Add storage. By default, the root volume is deleted on termination. 
6. You can add tags, which are key/value pairs that help you identify a feature. 
7. Configure security group. Defines how we SSH in the instance. Created `my-first-security-group`. 
8. Review and launch.
9. Create a key pair that will allow you to SSH in the machine. 

**SSH**. Control a remote machine from your command line. 

1. In the EC2 dashboard, select the instance and copy the IPv4 Public IP. 
2. Click on security groups  > view inbound rules to see which port is open and make sure your IP is authorized. 
3. `ssh -i EC2Tutorial.pem ec2-user@[IP]` . This will throw `WARNING: UNPROTECTED PRIVATE KEY FILE`. 
4. [**EXAM QUESTION**] To fix this, `chmod 0400 EC2Tutorial.pem`. 
5. Now we can repeat the SSH command and we are in the machine. 

**EC2 Instance Connect**. Browser based instance connection. Find it in the dashboard by clicking on your instance and then on Connect. It only works with the Amazon Linux 2 AMI. 

## Security Groups

Control how traffic is allowed into or out of EC2 machines. 

From the EC2 dashboard, on the left you can access security groups. For each security group we have Description, Inbound, Outbound and Tags. 

Inbound rules define which port is open and which IPs are allowed. Without inbound rules, we won't be able to access the instance. The IP `0.0.0.0/0` will allow SSH from anywhere. 

*Deeper dive*

Security groups act as firewalls on EC2 instances. They regulate:

* Access to Ports
* Authorised IP ranges - IPV4 and IPv6
* Control inbound network
* Control outbound network

Security groups: 

1. Can be attached to multiple instances
2. Are locked down to a region / VPC combination
3. Live outside of EC2. If traffic is blocked, the EC2 instance won't even see it. 
4. As a best practice, it's good to maintain one separate security group for SSH access. 
5. If your application is not accessible (time out) then it's a security group issue. 
6. But if you get a "connection refused" error, then it's an application error or the instance is not launched. 
7. All inbound traffic is blocked by default, and all outbound traffic is authorized. 

SGs can reference other SGs, for example an SG authorizing inbound for apps with Security Group 1 and Security Group 2. 

*Networking*

Two sorts of IPs. IPv4 and IPv6. 

IPv4: `1.160.10.240` . Format: 4 numbers in the 0-255 range. 

IPv6: 3ffe: `1900:4545:3:200:f8ff:fe21:67cf`

IPv4 is the most common format online. The other is newer and solves IOT problems. 

A private network range looks like this: `1.160.10.240/22`. All machines within the private network can communicate. 

Public IP must be unique across the whole web. It can be easily geolocated. 

Private IP means the machine can only be identified on the private network. Two different private networks can have the same IPs. Machines on a private network connect to WWW with an internet gateway that functions as a private proxy. 

Elastic IPs. When you stop and restart an EC2, it can change its public IP. If you need a fixed public IP, you need an Elastic IP. It can be attached to one machine only.

With an Elastic IP address, you can mask the failure of an instance or software by rapidly remapping the address to another instance in your account. 

You can only have 5 elastic IPs in your AWS account. Overall, it's best to avoid them and use DNS or Load Balancers. 

*Hands on*. By default, the EC2 machine has a private Ip for the internal AWS network and a public IP for WWW. We can only SSh into our EC2 machines through public IP. If the machine restarts, the public IP can change. 

You cannot connect to an instance with its Private IP (this is available only within AWS). If you stop and restart the instance, the public IP will change. You will then need to update your SSH command with the new IP. 

Under Network & Security, select Elastic IPs. Here you can allocate an Elastic IP address. Now the Public IP will not change if the instance is restarted. You can later release the Elastic IP address so you don't pay for it. 

*Launching an Apache Server on EC2*

We will install Apache Web Server and use it to display an `index.html`. 

1. Get the public IP and SSH into your instance. 
2. `sudo su ` to become root
3. `yum update -y` to update the machine.
4. `yum install -y httpd.x86_64` 
5. `systemctl start httpd.service` to start the service
6. `systemctl enable httpd.service` to enable across reboots
7. Apache Server starts on port 80. We need to open that port. 
8. Select security group. In the inbound rules select edit. Create a rule of type HTTP with TCPC on port 80 and IP `0.0.0.0/0`. 
9. Access the page at `http://[IP_ADDRESS]:80` and verify the server is active.
10. `echo "Hello World" > /var/www/html/index.html` to write in our index. 



*EC2 User Data*

We can boostrap our instances with an EC2 User data script. The script will only run once as the instance first starts. This is used to automate boot tasks. 

1. Terminate the instance. 
2. Create a new instance. In configure instance, go to Advanced Details. 
3. Paste the bash script in User data. This scripts will be automatically run with sudo. 

```bash
#!/bin/bash
# install httpd
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

4. Pass it the previous security group (`my-first-security-group`). It will open port 80. 
5. Select the existing key pair. 
6. Access the page at `http://[IP_ADDRESS]:80`. 
7. Access the machine with `ssh -i EC2Tutorial.pem ec2-user@[IP]`.



**#quizzes**

You pay for an EC2 instance **compute** component --> only when it's in "running" state.

You are getting a permission error exception when trying to SSH into your Linux Instance --> the key is missing permissions chmod 0400

You are getting a network timeout when trying to SSH into your EC2 instance --> your security groups are misconfigured

Security groups can reference all the following except --> DNS name (the others were: IP address, CIDR block, Security Group).