## Decision: SSM Session Manager over SSH

When setting up EC2 access, I initially started with an SSH bastion host. AWS recommended SSM Session Manager instead. SSM eliminates the need to create and manage SSH keys, which reduces the risk of keys being lost or exposed. It also means no inbound port 22 needs to be open on the EC2 instance, which removes an attack surface. Access is controlled through IAM roles and policies instead.

---

## Decision: VPC CIDR 10.0.0.0/16

This CIDR was recommended in the project checklist. At the time of Build 1 I followed it without fully understanding why. What I know now is that 10.0.0.0 is a private IP range that doesn't route on the public internet, and /16 gives the VPC 65,536 IP addresses large enough to support multiple subnets across availability zones without running out of space.

---

## Decision: Two Availability Zones

Resources were deployed across two availability zones to provide high availability and fault tolerance. So if incase one AZ experiences an outage, traffic and workloads can continue running in the second AZ without interruption. I thought just sticking with two AZs was the most cost effective approach for this enough redundancy for a production-style setup without the added expense of a third AZ.

---

## Decision: Public vs Private Subnets

The network was built with 2 public and 2 private subnets across both availability zones. Public subnets are for the resources that need to be directly reachable from the internet such as the load balancer and NAT Gateway. Private subnets are for the resources that should never be directly exposed like the EC2 web servers. This separation limits the attack surface. Even if something in the public subnet is compromised, the private subnet adds another layer of protection that an attacker would have to get through.
