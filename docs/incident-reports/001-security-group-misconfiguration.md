# Incident Report 001: Security Group Misconfiguration

## Summary

An unauthorized inbound rule was intentionally added to sg-ec2-web opening port 22 (SSH) to 0.0.0.0/0 as part of a break and fix exercise. The change was detected through CloudTrail event history. The misconfiguration exposed the EC2 instances to potential SSH access from anywhere on the internet, bypassing the defense in depth design of the security group architecture. The rule was identified and removed within the same session.

## Timeline

- **14:23** — Inbound SSH rule (port 22, source 0.0.0.0/0) added to sg-ec2-web
- **14:23** — CloudTrail logged the event: AuthorizeSecurityGroupIngress
- **15:39** — SSH rule identified and removed from sg-ec2-web
- **15:39** — Security group restored to original configuration

## Root Cause

An inbound rule was manually added to sg-ec2-web by an IAM user with administrative access. The rule opened port 22 (SSH) to all internet traffic (0.0.0.0/0) which was not part of the intended security group configuration. In a real world scenario this could result from human error, a miscommunication during a change request, or a malicious insider with elevated permissions.

## Impact and Blast Radius

Opening port 22 to 0.0.0.0/0 exposed the EC2 instances to direct SSH access attempts from anywhere on the internet, bypassing the ALB and sg-ec2-web's intended traffic controls. 

If an attacker gained SSH access to an EC2 instance they could potentially:
- Plant malware or exfiltrate data from the web server
- Attempt to access the IAM role credentials attached to the instance
- Use the compromised instance as a launching point to probe other resources in the VPC

The blast radius is limited by the IAM role attached to the instance — EC2-WebServer-Role only has SSM and CloudWatch logging permissions. However the instance itself and any data on it would be at risk.

Two EC2 instances were exposed — one in each private subnet.

## Remediation

After identifying the unauthorized inbound rule via CloudTrail, the rule was removed by navigating to VPC → Security Groups → sg-ec2-web and deleting the port 22 inbound rule. The security group was restored to its original configuration allowing only HTTP traffic from sg-alb. The fix was completed at 15:39 and verified by reviewing the updated inbound rules.

## Prevention

To prevent this type of misconfiguration in the future the following controls could be implemented:

- **Restrict IAM permissions** — limit which IAM users and roles have the ability to modify security groups. Not everyone who has AWS access needs the ability to change network security rules.
- **AWS Config rules** — set up automated rules that detect when a security group allows unrestricted access on sensitive ports like 22 or 3389 and flag it immediately.
- **CloudWatch alerts on CloudTrail** — create a metric filter that triggers an SNS alert any time a security group inbound rule is modified. This would have alerted immediately at 14:23 rather than requiring manual investigation.

## Lessons Learned

- Detecting security group changes in CloudTrail requires knowing what the ports do. 
If you don't know that port 22 is SSH and that it shouldn't be open to 0.0.0.0/0 
you wouldn't recognize it as a threat even if you saw it in the logs. 
Port knowledge is a prerequisite for effective monitoring.

- CloudTrail captures events but does not alert by default. In a real SOC environment 
a CloudWatch metric filter on CloudTrail logs would trigger an SNS alert whenever 
a security group is modified. This project does not have that configured yet — 
something to improve in a future build.

- One rule change was enough to break the entire defense in depth design. 
The layered security model only works if every layer is maintained correctly. 
Human error or malicious insider activity at any layer can compromise the whole system.

- This exercise reinforced the importance of studying common ports and their 
associated attack vectors so that anomalies in logs and configurations 
are recognizable immediately.
