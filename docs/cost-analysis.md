# Cost Analysis

## Month-to-Date Cost Breakdown (August 2026)

| Service | Monthly Cost | Notes |
|---------|--------------|-------|
| EC2-Other (NAT Gateway) | $16.19 | Biggest cost driver — NAT Gateway hourly charge |
| Elastic Load Balancing | $7.27 | ALB fixed hourly cost |
| EC2-Instances | $6.68 | 2x t3.micro instances |
| VPC | $4.97 | Elastic IP charges |
| S3 | $0.03 | CloudTrail log storage |
| CloudTrail | $0.00 | Within free tier |
| GuardDuty | $0.00 | Within free tier |
| CloudWatch | $0.00 | Within free tier |
| **TOTAL** | **$35.13** | 

---

## Cost Observations

The NAT Gateway is the biggest cost driver at $16.19 — and this is with me deleting and recreating it between sessions to manage cost. If left running continuously it would cost approximately $32/month on its own.

To manage cost during this build I made the decision to only run the NAT Gateway during active build sessions and delete it afterward. This cut the NAT Gateway cost roughly in half but added setup time at the start of each session.

The ALB at $7.27 is a fixed cost — it charges hourly regardless of traffic. The EC2 instances at $6.68 are also hourly but relatively cheap at t3.micro pricing.

Total spend for the month is $35.13 which is within my $50 budget, though the project is not yet complete for the month.

---

## Cost Optimization Opportunities

**NAT Gateway**
The NAT Gateway is the most expensive resource in this build. In a production environment a few alternatives worth considering:

- VPC Endpoints for services like S3 and Systems Manager — traffic stays within AWS network and avoids NAT Gateway charges entirely
- If high availability is required, a second NAT Gateway in the second AZ adds ~$32/month but eliminates a single point of failure
- For this learning project, deleting the NAT Gateway between sessions was the most practical cost control

**EC2 Instances**
Currently running t3.micro which is free tier eligible. Could be downsized to t3.nano to save further but t3.micro was chosen to stay within free tier limits.

**Reserved Instances**
At current on-demand pricing this setup costs ~$35-72/month depending on NAT Gateway uptime. A 1-year Reserved Instance commitment on the EC2 instances could reduce that cost by roughly 30-40%.

## Production Cost Projection

If this infrastructure were running in production with full uptime:
- NAT Gateway (x2 for HA): ~$64/month
- ALB: ~$16/month
- EC2 (t3.micro x2): ~$15/month
- EBS Storage: ~$2/month
- **Estimated production total: ~$100-120/month**
