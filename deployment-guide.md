# Deployment Guide

## Prerequisites

- AWS account
- Access to AWS CloudFormation
- Permission to create VPC, EC2, RDS, IAM and ELB resources
- AWS region selected

## Deployment

1. Sign in to the AWS Management Console.
2. Open CloudFormation.
3. Select Create Stack.
4. Choose "Upload a template file."
5. Upload the CloudFormation YAML template.
6. Enter the required database password.
7. Review the configuration.
8. Acknowledge IAM resource creation.
9. Create the stack.
10. Monitor the Events tab until the stack reaches CREATE_COMPLETE.

## Verification

After deployment:

- Verify the EC2 instance is running.
- Verify the target group reports the EC2 instance as healthy.
- Verify the Application Load Balancer is active.
- Verify the RDS database is available.
- Open the ALB DNS name in a browser.

