# Troubleshooting Notes

## Target Group Health Check Failure

### Problem
The EC2 instance initially appeared unhealthy in the Application
Load Balancer target group.

### Investigation
I checked:

- EC2 security group rules
- Target group port
- HTTP health check settings
- Apache web server status
- Application files in /var/www/html

### Resolution
I corrected the configuration so that the EC2 server could respond
successfully to the ALB health check on port 80.

### What I Learned
A load balancer will only forward traffic to targets that pass its
configured health checks.