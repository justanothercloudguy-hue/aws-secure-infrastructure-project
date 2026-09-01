# Incident Report 002: Overprivileged IAM Role

## Summary

AdministratorAccess policy was intentionally attached to EC2-WebServer-Role as part of a break and fix exercise. This gave any EC2 instance using this role full administrative access to the entire AWS account, putting the entire infrastructure at risk. The change was detected through CloudTrail event history and the policy was removed within 11 minutes of being attached.

## Timeline

- **11:11 AM (15:11 UTC)** — AdministratorAccess policy attached to EC2-WebServer-Role by IAM user shawn-admin
- **11:11 AM (15:11 UTC)** — CloudTrail logged the event: AttachRolePolicy
- **11:22 AM** — AdministratorAccess policy identified and removed from EC2-WebServer-Role
- **11:22 AM** — Role restored to original permissions: AmazonSSMManagedInstanceCore and CloudWatch Logs only

## Root Cause

The AdministratorAccess policy was manually attached to EC2-WebServer-Role by an IAM user with sufficient permissions to modify roles. In a real world scenario this type of change could result from an insider threat attempting to escalate privileges quietly — attaching permissions to an existing role rather than creating a new one to avoid drawing attention. It could also result from human error during a troubleshooting session where someone grants broad permissions temporarily and forgets to remove them. Either way the root cause is insufficient controls around who can modify IAM roles and what policies can be attached.

## Impact and Blast Radius

With AdministratorAccess attached to EC2-WebServer-Role any compromised EC2 instance would have near-unlimited access to the entire AWS account. A malicious actor could:

- Spin up additional resources to run up costs 
- Shut down or delete existing infrastructure
- Access or exfiltrate sensitive data from any S3 bucket or other storage
- Create new IAM users or roles to establish persistent backdoor access

## Remediation

## Prevention

## Lessons Learned
