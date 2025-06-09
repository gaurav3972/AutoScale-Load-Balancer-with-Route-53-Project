# 🏗️ Scalable Web Application Architecture on AWS
## Using EC2 + ALB + Auto Scaling Group + Route 53

## 🎯 Learning Objectives
* Create a Launch Template for EC2 instances.

* Set up an Auto Scaling Group (ASG) to manage EC2 instances based on demand.

* Configure an Application Load Balancer (ALB) to distribute traffic.

* Integrate Amazon Route 53 with the ALB using a custom domain name.

* Ensure high availability and fault tolerance of the application.
****
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/1.png)
## ✅ Prerequisites
You should be familiar with:

* EC2 Basics – including launching instances and connecting to them using SSH

* Working with the AWS Console

* Auto Scaling Groups (ASG) and Launch Templates

* CloudWatch – understanding metrics and alarms

* EC2 Security Groups – managing firewall rules

* Route 53 – understanding domain registration, hosted zones, and routing traffic using record sets
****
## 🛠️ Step 1: Login to AWS Console
Navigate to https://aws.amazon.com

Sign in with your IAM or root credentials.
****
## 🌐 Step 2: Create an Application Load Balancer (ALB)
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-07%20213238.png)
1. Open the EC2 Dashboard.

2. In the left menu, go to Load Balancing > Load Balancers > Click Create Load Balancer.

3. Choose Application Load Balancer.

4. Configure:

* Name: web

* Scheme: Internet-facing

* IP address type: **IPv4**

5. Network Mapping:

* Select all available subnets except us-west-2d.

6. Security Group:

* Click Create a new security group

* Name: web

* Description: ELB for a webserver cluster

* Add inbound rule: HTTP from Anywhere-IPv4.
****
## 🎯 Step 2.1: Create and Configure a Target Group
A Target Group defines which EC2 instances receive traffic from your ALB.
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-07%20213218.png)
1. During ALB creation, in the Listeners and routing section, click Create target group.

2. Configure the target group:

* Target type: Instances

* Name: web

* Protocol: HTTP

* Port: **80**

* VPC: Select your VPC.

3. Click Next.

4. Skip registering targets manually (Auto Scaling Group will handle it).

5. Click Create target group.

6. Return to ALB creation, refresh the Default action dropdown, and select your new target group (web).

7. Click Create load balancer.

## 🧪 Monitoring Target Group Health
To monitor your target group health:

1. Go to EC2 Dashboard > Target Groups.

2. Select your target group (web).

3. Click on the Targets tab.

4. **Verify the Health status of registered instances. They should be healthy once passing health checks.**

**If targets show unhealthy:**

* Confirm security group rules **allow HTTP (port 80) inbound.**

* Ensure Apache (httpd) service is running and serving content.

* Verify health check path (default /) is accessible.
****
## 🧱 Step 3: Create a Launch Template
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-07%20213320.png)
1. Go to EC2 > Security Groups > Click Create Security Group:

* Name: webserver-cluster

* Inbound rules:

* SSH from Anywhere-IPv4

* HTTP from Anywhere-IPv4

2. Navigate to Launch Templates > Click Create launch template:

* Name: webserver-cluster

* Description: Lab launch template

* **AMI: Amazon Linux 2**

* **Instance Type: t2.micro**

* Key Pair: Use existing key

* Security Group: webserver-cluster

* Monitoring: Enable Detailed CloudWatch Monitoring

* User Data:



`#!/bin/bash
sudo amazon-linux-extras install epel -y
sudo yum install -y httpd php
sudo systemctl start httpd
sudo yum install -y stress`

3. Click Create launch template.
****
## 📈 Step 4: Create an Auto Scaling Group (ASG)
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-07%20213205.png)
1. Navigate to Auto Scaling Groups > Click Create.

2. Configure:

Name: webserver-cluster

Launch Template: webserver-cluster

3. Network: Select all subnets except us-west-2d.

4. Load Balancing:

* **Attach to existing target group: web | HTTP**

* **Enable ELB Health Checks (Grace period: 120 seconds)**

5. Monitoring: Enable group metrics collection in CloudWatch.

6. Group Size:

Min: 1

Desired: 1

Max: 5

7. Scaling Policy:

* Type: Target Tracking

* Policy Name: Lab scaling policy

* Metric: Average CPU Utilization

* Target: 80%

* Instance Warmup: 0

8. Click Create Auto Scaling Group.
****
## 🔍 Step 5: Test Auto Scaling Behavior
1. Navigate to EC2 > Instances > Select an instance launched by the ASG.

2. Click Connect using EC2 Instance Connect.

3. Run a stress test to trigger scale-out:

`
stress --cpu 2 --io 1 --vm 1 --vm-bytes 128M --timeout 5m`
4. In the AWS Console, check:

* Monitoring tab > View in metrics for CPU spikes.

* Auto Scaling Groups > Activity tab for scaling events.
****

## 🌍 Step 6: Configure Route 53 for Domain Integration
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-08%20192835.png)
1. Open Route 53 Console.

2. Click Hosted Zones > Create Hosted Zone:

Domain Name: patilenterprise.shop

3. Once created, click Create Record:

Record Type: A – IPv4 address

**Alias: Yes**

**Alias Target: Select your ALB’s DNS name.**

4. Save the record and wait a few minutes.

5. Access your domain in a browser: http://patilenterprise.shop should display the Apache welcome page served by your EC2 instances.
![](https://github.com/gaurav3972/AutoScale-Load-Balancer-with-Route-53-Project/blob/main/images/Screenshot%202025-06-08%20192946.png)
Sure! Here’s a **short version** of the important tips:

---

## ⚠️ Important Tips (Short Version)

1. **Security Groups:**
   Allow HTTP from anywhere to ALB, SSH only from your IP to instances.

2. **Health Checks:**
   Use HTTP health checks on target group; ensure instances serve the correct path.

3. **Auto Scaling:**
   Use target tracking policies (CPU) with sensible min/max instance limits.

4. **Launch Templates:**
   Use launch templates for flexibility; update when needed.

5. **User Data:**
   Automate instance setup with user data scripts.

6. **Load Balancer:**
   Use Internet-facing ALB; monitor with access logs.

7. **Route 53:**
   Create alias A record pointing to ALB; DNS changes may take minutes.

8. **Cost:**
   Monitor usage; scale down when idle.

9. **Monitoring:**
   Use CloudWatch for metrics and alerts.

10. **Backup:**
    Backup data and create AMIs regularly.
    Here’s a short **Example Scenario** for your setup:

---

## 🔍 Example Scenario (Short)

You run a website `patilenterprise.shop` serving customers globally. To ensure high availability and handle traffic spikes, you set up:

* An **Application Load Balancer** (ALB) to distribute incoming HTTP requests across multiple EC2 instances.
* A **Target Group** linked to the ALB that monitors instance health.
* An **Auto Scaling Group** that automatically adds or removes EC2 instances based on CPU usage.
* **Launch Templates** that define how each instance is configured (Apache web server installed automatically).
* **Route 53** DNS pointing your domain to the ALB.

When traffic increases, the Auto Scaling Group launches more instances; when traffic drops, it scales down — all transparent to users, ensuring smooth performance and cost efficiency.
Sure! Here’s the summary in paragraph form:

---
## 📋 Summary
This project guides you through building a scalable and highly available web application infrastructure on AWS. It starts with creating an Application Load Balancer (ALB) that distributes incoming HTTP traffic across multiple EC2 instances. A Target Group monitors the health of these instances to ensure traffic is routed only to healthy servers. Using a Launch Template, EC2 instances are configured with Apache, PHP, and monitoring tools. An Auto Scaling Group (ASG) automatically adjusts the number of running instances based on CPU utilization, ensuring your application can handle varying traffic loads efficiently. Finally, Route 53 is configured to map your custom domain (`patilenterprise.shop`) to the ALB, providing seamless access to users. This architecture ensures your web app remains resilient, scalable, and highly available with minimal manual management.
****
Here’s a concise **License** section you can add to your document:

---

### 🔐 License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT). You are free to use, modify, and distribute this work with proper attribution. No warranty is provided.

---

