# Security Controls

---

## Security Group Strategy

Traffic flows from the internet through the Internet Gateway to the ALB, which then distributes it to the EC2 instances. At each step a security group controls what's allowed through.

sg-alb governs the load balancer — it accepts HTTP and HTTPS traffic from anywhere on the internet. sg-ec2-web governs the EC2 instances — it only accepts traffic that came from sg-alb, nothing else. This creates multiple layers of protection before any request reaches the actual web server. Even if someone bypasses the ALB they still can't reach the EC2 directly.

sg-database is in place for future use — it will only accept connections from sg-ec2-web, continuing the same layered approach.

---

## Principle of Least Privilege

In Week 2 I created an IAM Role for EC2 called EC2-WebServer-Role using the principle of least privilege. I attached the AmazonSSMManagedInstanceCore managed policy and added a custom inline CloudWatch Logs policy.

The role only has two permissions, the minimum needed to communicate with SSM Session Manager and the ability to create and write logs to CloudWatch. Nothing else. The EC2 instance cannot touch S3, modify IAM, or spin up other resources. If the instance were ever compromised, the blast radius is limited to what that role can do — which is almost nothing outside of its intended function.

---

## Defense in Depth

During this build I started to understand that security isn't just one thing you bolt on at the end — it lives between and on top of every layer of the infrastructure. 

Starting from the beginning: MFA on both root and IAM accounts, a dedicated IAM user for daily use instead of root, a VPC that isolates resources from the public internet, public and private subnets that separate what's exposed from what's protected, security groups that control exactly what traffic can move between layers, an IAM role with only the minimum permissions the EC2 instances need, and SSM instead of SSH so there's no exposed port 22.

No single layer is the security. All of them together are the security. If one layer fails, the next one is still there.

---

## Blast Radius Analysis

As someone still learning, I needed to understand what would happen if I misconfigured something or if part of this infrastructure was compromised. Here's my understanding so far of the blast radius at each layer.

The ALB is governed by `sg-alb` a security group which I configured to only allow HTTP and HTTPS traffic from the internet. If I had misconfigured this and opened it to all ports it would expose the load balancer to traffic it shouldn't be receiving and potentially create a path to the EC2 instances. This is the reasoning for keeping those inbound rules tight.

The EC2 instances are governed by `sg-ec2-web` a security group which only accepts traffic coming from `sg-alb`. Even if someone got past the ALB the security group would still block direct access to the instances. 

What I also came to understand is that any compromise to an EC2 instance can give an attacker access to the same permissions the attached IAM role has. In this setup `EC2-WebServer-Role` only has access to SSM Session Manager and CloudWatch logging — nothing else. AWS only allows one IAM role per EC2 instance at a time which forces deliberate decision making about what permissions each instance actually needs. This raises a question I want to explore further — how do you handle situations where different applications on the same instance need different levels of access?

From what I understand the public and private subnets live on separate routing planes. Getting into the public subnet doesn't automatically give someone access to the private subnet where the EC2 instances are. The only intended path is through the load balancer.

I don't yet fully understand every attack vector that could exist in this setup but this is my current thinking. I expect my understanding of blast radius to deepen as I continue building and studying.

---

## Monitoring and Logging

### What I'm Logging and Why

Three monitoring services are active in this build:

**CloudTrail** captures every API call and console action made in the AWS account — who did what, when, and from where. This is what allowed me to detect both break and fix scenarios in this project. Without CloudTrail there would be no record of the security group change or the IAM role modification.

**CloudWatch** monitors the health and performance of resources — CPU utilization, instance status checks, and ALB unhealthy host counts. Alarms are configured to trigger SNS notifications when thresholds are breached. This removes the need to manually check each service and creates a centralized view of infrastructure health.

**GuardDuty** provides threat detection by analyzing CloudTrail logs, VPC flow logs, and DNS logs for suspicious behavior. It runs in the background passively — no configuration needed beyond enabling it.

Together these three services mean nothing happens in this infrastructure without a record of it. That's the foundation of operational visibility.

---

### What Alerts I've Set Up

Three CloudWatch alarms are configured, all connected to an SNS topic that sends email notifications:

**Unhealthy Hosts (ALB)** — triggers when the load balancer detects one or more EC2 instances failing health checks. This is why two instances are running across two AZs — if one goes down the alarm fires while the second instance continues serving traffic.

**Status Check Failed** — monitors the EC2 instance itself for system level failures. Where the unhealthy hosts alarm watches from the ALB's perspective, this alarm watches the instance directly. They complement each other — one catches network level issues, the other catches instance level issues.

**High CPU** — triggers when CPU utilization exceeds 80% for two consecutive 5 minute periods. In a production environment this would signal the need to scale up or investigate a runaway process.

---

### How I'd Respond to Each Alert

**Unhealthy Hosts** — investigate which instance failed its health check, check CloudWatch logs for errors, attempt to restart the instance. If it doesn't recover launch a replacement.

**Status Check Failed** — check the instance status in EC2 console, review system logs via SSM Session Manager, reboot the instance if unresponsive.

**High CPU** — connect via SSM and investigate running processes, determine if it's a legitimate traffic spike or a runaway process, scale up if needed.

In a real environment each of these would follow a documented incident response playbook with defined escalation paths. That's something to build out in a future iteration of this project.

---

### Cost Breakdown of Monitoring Tools

| Service | Monthly Cost | Notes |
|---------|--------------|-------|
| CloudTrail | $0.00 | Within free tier for this project size |
| CloudWatch | $0.00 | Within free tier for this project size |
| GuardDuty | $0.00 | Within free tier for this project size |
| SNS | $0.00 | Within free tier |
| **Total** | **$0.00** | All monitoring services running at no additional cost |

One of the advantages of this monitoring setup is that all three core services — CloudTrail, CloudWatch, and GuardDuty — fall within AWS free tier limits for a project of this size. In a production environment with higher log volumes and more resources these costs would increase but remain relatively low compared to the value they provide.
