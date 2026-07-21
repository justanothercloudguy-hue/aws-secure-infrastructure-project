## Decision: SSM Session Manager over SSH

When setting up EC2 access, I initially started with an SSH bastion host. AWS recommended SSM Session Manager instead. SSM eliminates the need to create and manage SSH keys, which reduces the risk of keys being lost or exposed. It also means no inbound port 22 needs to be open on the EC2 instance, which removes an attack surface. Access is controlled through IAM roles and policies instead.

---

## Decision: VPC CIDR 10.0.0.0/16

This CIDR was recommended in the project checklist. At the time of Build 1 I followed it without fully understanding why. What I know now is that 10.0.0.0 is a private IP range that doesn't route on the public internet, and /16 gives the VPC 65,536 IP addresses — large enough to support multiple subnets across availability zones without running out of space.
