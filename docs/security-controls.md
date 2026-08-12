# Security Controls

---

## Security Group Strategy

Traffic flows from the internet through the Internet Gateway to the ALB, which then distributes it to the EC2 instances. At each step a security group controls what's allowed through.

sg-alb governs the load balancer — it accepts HTTP and HTTPS traffic from anywhere on the internet. sg-ec2-web governs the EC2 instances — it only accepts traffic that came from sg-alb, nothing else. This creates multiple layers of protection before any request reaches the actual web server. Even if someone bypasses the ALB they still can't reach the EC2 directly.

sg-database is in place for future use — it will only accept connections from sg-ec2-web, continuing the same layered approach.
