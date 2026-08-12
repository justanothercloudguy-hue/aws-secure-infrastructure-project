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
