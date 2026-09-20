# AWS Resource Cleanup

This project was deployed for learning and portfolio purposes.

After testing was completed, AWS resources were intentionally
decommissioned to prevent unnecessary cloud charges.

## CloudFormation Resources

The CloudFormation stack was deleted after testing.

## Resources Checked During Cleanup

I verified removal of resources that could generate charges,
including:

- EC2 instances
- Application Load Balancer
- Target Groups
- RDS database
- NAT Gateway
- Elastic IP address
- VPC networking resources

## Why Cleanup Matters

Cloud resources can continue generating costs even when they are
not actively being used.

Deleting unused infrastructure is part of responsible cloud
resource and cost management.