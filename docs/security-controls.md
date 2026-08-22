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

## Defense in Depth**

During this build I started to understand that security isn't just one thing you bolt on at the end — it lives between and on top of every layer of the infrastructure. 

Starting from the beginning: MFA on both root and IAM accounts, a dedicated IAM user for daily use instead of root, a VPC that isolates resources from the public internet, public and private subnets that separate what's exposed from what's protected, security groups that control exactly what traffic can move between layers, an IAM role with only the minimum permissions the EC2 instances need, and SSM instead of SSH so there's no exposed port 22.

No single layer is the security. All of them together are the security. If one layer fails, the next one is still there.

---

---

## Blast Radius Analysis**

As someone still learning, I needed to understand what would happen if I misconfigured something or if part of this infrastructure was compromised. Here's my understanding so far of the blast radius at each layer.

The ALB is governed by `sg-alb` a security group which I configured to only allow HTTP and HTTPS traffic from the internet. If I had misconfigured this and opened it to all ports it would expose the load balancer to traffic it shouldn't be receiving and potentially create a path to the EC2 instances. This is the reasoning for keeping those inbound rules tight.

The EC2 instances are governed by `sg-ec2-web` a security group which only accepts traffic coming from `sg-alb`. Even if someone got past the ALB the security group would still block direct access to the instances. 

What I also came to understand is that any compromise to an EC2 instance can give an attacker access to the same permissions the attached IAM role has. In this setup `EC2-WebServer-Role` only has access to SSM Session Manager and CloudWatch logging — nothing else. AWS only allows one IAM role per EC2 instance at a time which forces deliberate decision making about what permissions each instance actually needs. This raises a question I want to explore further — how do you handle situations where different applications on the same instance need different levels of access?

From what I understand the public and private subnets live on separate routing planes. Getting into the public subnet doesn't automatically give someone access to the private subnet where the EC2 instances are. The only intended path is through the load balancer.

I don't yet fully understand every attack vector that could exist in this setup but this is my current thinking. I expect my understanding of blast radius to deepen as I continue building and studying.

---
