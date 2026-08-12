## Decision: SSM Session Manager over SSH

When setting up EC2 access, I initially started with an SSH bastion host. AWS recommended SSM Session Manager instead. SSM eliminates the need to create and manage SSH keys, which reduces the risk of keys being lost or exposed. It also means no inbound port 22 needs to be open on the EC2 instance, which removes an attack surface. Access is controlled through IAM roles and policies instead.

---

## Decision: VPC CIDR 10.0.0.0/16

This CIDR was recommended in the project checklist. At the time of Build 1 I followed it without fully understanding why. What I know now is that 10.0.0.0 is a private IP range that doesn't route on the public internet, and /16 gives the VPC 65,536 IP addresses large enough to support multiple subnets across availability zones without running out of space.

---

## Decision: Two Availability Zones

Resources were deployed across two availability zones to provide high availability and fault tolerance. So if incase one AZ experiences an outage, traffic and workloads can continue running in the second AZ without interruption. I thought just sticking with two AZs was the most cost effective approach for this enough redundancy for a production style setup without the added expense of a third AZ.

---

## Decision: Public vs Private Subnets

The network was built with 2 public and 2 private subnets across both availability zones. Public subnets are for the resources that need to be directly reachable from the internet such as the load balancer and NAT Gateway. Private subnets are for the resources that should never be directly exposed like the EC2 web servers. This separation limits the attack surface. Even if something in the public subnet is compromised, the private subnet adds another layer of protection that an attacker would have to get through.

---

## Decision: NAT Gateway

A NAT Gateway was created in the public subnet to give private subnet resources outbound access to the internet for things like software updates without exposing them to inbound internet traffic.

To manage cost, the NAT Gateway is only created during active build sessions and deleted afterward. At $0.045/hour it's the biggest cost driver in this project. The first two builds were used to confirm it functioned correctly. The plan going forward is to continue creating and deleting it per session until the full build is complete and ready to leave running.

---

## Decision: EC2 Instances in Private Subnets

I put the EC2 instances in private subnets to reduce exposure from attacks. By doing this the private subnets add an additional layer of security the only way in is through the load balancer, which gives me one controlled entry point. This in turn limits the attack surface and makes it harder for anyone trying to access sensitive information directly.


---

## Decision: ALB over NLB

I chose an Application Load Balancer over a Network Load Balancer because this project is serving web traffic HTTP requests to a website. ALB operates at the application level and understands HTTP, which makes it the right fit for routing web traffic to EC2 instances. NLB operates at the network level and is built for high performance, low latency use cases like gaming or streaming where raw speed matters more than application level routing. Using NLB for a web application would be overkill for what this project requires. I don't yet fully understand every scenario where NLB would be the right choice over ALB that's something I plan to go deeper on as I continue building this project.

---

## Decision: Security Groups Reference Each Other by ID**

I chose to reference security groups by ID rather than by IP address because IP addresses change. In this project alone I've been terminating and rebuilding the NAT Gateway every session — each time it gets a new IP. If I had used IP addresses in my rules they would break every single time. By referencing sg-alb directly in sg-ec2-web, I don't have to update anything when a resource gets rebuilt. The rule follows the security group not the IP. This same thinking applies in real world scenarios like replacing a load balancer during an upgrade — the IP changes but the security group stays attached and traffic keeps flowing.

---
