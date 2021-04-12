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





