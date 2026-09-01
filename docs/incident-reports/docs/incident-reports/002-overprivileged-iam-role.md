# Incident Report 002: Overprivileged IAM Role

## Summary

AdministratorAccess policy was intentionally attached to EC2-WebServer-Role as part of a break and fix exercise. This gave any EC2 instance using this role full administrative access to the entire AWS account, putting the entire infrastructure at risk. The change was detected through CloudTrail event history and the policy was removed within 11 minutes of being attached.

## Timeline

- **11:11 AM (15:11 UTC)** — AdministratorAccess policy attached to EC2-WebServer-Role by IAM user shawn-admin
- **11:11 AM (15:11 UTC)** — CloudTrail logged the event: AttachRolePolicy
- **11:22 AM** — AdministratorAccess policy identified and removed from EC2-WebServer-Role
- **11:22 AM** — Role restored to original permissions: AmazonSSMManagedInstanceCore and CloudWatch Logs only

## Root Cause

## Impact and Blast Radius

## Remediation

## Prevention

## Lessons Learned
