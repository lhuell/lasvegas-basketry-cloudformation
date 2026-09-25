# Basketry Terraform Lab — Progress README

**Status:** Network, app server, load balancer, and database deployed; test order saved directly in MySQL. The browser order form has **not** been created yet.

**AWS Region:** `us-east-2` (Ohio)  
**Local project folder:** `D:\1Terraform\Main.tf\DBasketry-Terraform-Fresh`  
**Terraform configuration:** `Main.tf`  
**Last verified:** September 24, 2026

This is a hands-on learning environment for the Basketry local gift-ordering concept. It currently serves a health-check page over HTTP. Use made-up test details only; it is not ready to collect real customer orders.

## Architecture

```text
Browser → public Application Load Balancer → private EC2 (Apache/PHP)
                                                  │
                                                  └→ private RDS MySQL

Private EC2 → NAT gateway → outbound AWS services and package repositories
```

The ALB uses public subnets in two Availability Zones. The app instance is in `private_a`. RDS uses a subnet group containing `private_a` and `private_b`. Security groups allow HTTP to the ALB, HTTP from the ALB to the app, and MySQL from the app to RDS.

## Work completed

| Step | What we did | Purpose / verification |
| --- | --- | --- |
| 1. Local tools | Installed Terraform, Git, and AWS CLI on Windows; used VS Code PowerShell in the project folder and signed in to AWS. | `terraform init` and AWS authentication worked. The AWS CLI region was corrected from `east-2` to `us-east-2`. |
| 2. Terraform configuration | Created `Main.tf` with the AWS provider set to `us-east-2`. | Terraform can plan changes for Ohio. |
| 3. VPC | Deployed `basketry-terraform-lab-vpc`, CIDR `10.20.0.0/16`. | Provides a separate lab network. |
| 4. Subnets | Added public `10.20.101.0/24` and private `10.20.1.0/24` in `us-east-2a`; public `10.20.102.0/24` and private `10.20.2.0/24` in `us-east-2b`. | Supports separate public and private tiers across two Availability Zones. |
| 5. Public routing | Added an internet gateway, public route table with `0.0.0.0/0` to the gateway, and associations for both public subnets. | Gives the ALB subnets internet routing. |
| 6. Private routing | Added a private route table and associated both private subnets. | Keeps their routes separate from the public tier. |
| 7. Outbound routing | Added one Elastic IP, one NAT gateway in `public_a`, and a `0.0.0.0/0` route from the private route table to the NAT gateway. | Lets the private EC2 server reach AWS services and install packages. This gateway incurs ongoing charges. |
| 8. Security groups | Added ALB HTTP ingress from the internet, app HTTP ingress only from the ALB group, and DB MySQL port 3306 ingress only from the app group. | Restricts traffic along the intended path. |
| 9. EC2 role | Created an EC2 IAM role, attached `AmazonSSMManagedInstanceCore`, and created an instance profile. | Enables Session Manager access to the private app server. |
| 10. App server | Launched one Amazon Linux 2023 `t3.micro` in `private_a` without a public IP. User data installed Apache/PHP and created a health page. | In Session Manager, `curl http://localhost` returned `Basketry Terraform app server is running`. |
| 11. Load balancer | Created an internet-facing ALB, port 80 listener, HTTP target group, and EC2 target attachment. | Opening the ALB DNS name in a browser displayed the app health page. The ALB currently uses HTTP, not HTTPS. |
| 12. RDS subnet group | Included both private subnets. | Meets the two-Availability-Zone subnet group requirement. |
| 13. Database | Created a private, encrypted RDS MySQL `db.t3.micro` with a managed master password in Secrets Manager. | Connected from EC2 through Session Manager to `basketry_orders`. |
| 14. SQL test | Manually created the `orders` table in MySQL, inserted one made-up `Happy Birthday` test order, and selected it. | The query returned one saved test row. **The SQL schema and row were created manually, outside Terraform.** |
| 15. Secret access | Added a Terraform IAM role policy granting the EC2 role `secretsmanager:GetSecretValue` on this RDS secret. | `aws secretsmanager get-secret-value --query ARN` worked both as the Session Manager shell user and as the `apache` user, without displaying the password. |

## Current test page

The Terraform output `basketry_alb_dns_name` gives the ALB hostname. The home page currently displays:

> Basketry Terraform app server is running

There is **no browser order form yet**. The `orders` table currently has only the synthetic row inserted through the MySQL client. The proposed `/order.php` step was paused before creating the file.

## Where to resume

1. Open the same project folder in VS Code. Keep `Main.tf` and `terraform.tfstate` together. Run Terraform in the VS Code PowerShell terminal in that folder.
2. If AWS authentication has expired, run `aws login`. Check the intended account and region before making changes.
3. In EC2 Session Manager, add the test `/order.php` page to `/var/www/html/` and run `php -l /var/www/html/order.php` to check syntax. The page should retrieve the RDS secret using the EC2 role and insert orders using a prepared SQL statement. Keep the existing `/` health page in place.
4. Open `/order.php` through the ALB, submit a **made-up** test order, then verify the new row with `SELECT` in MySQL.
5. Record the application code and database schema in the project so a fresh deployment can recreate them. A manual edit on EC2 and a manual `CREATE TABLE` are not automated by the current Terraform configuration.
6. Continue with the planned S3 image bucket and access rules; add HTTPS with a domain and certificate if you decide to use this beyond the HTTP lab.
7. At project completion, prepare the requested final table of all steps, tests, and cleanup results.

## Everyday Terraform commands

Run these in **VS Code PowerShell**, from the folder containing `Main.tf`:

```powershell
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
```

`plan` previews changes. `apply` shows a new plan and requests `yes` before changing AWS. Always read whether a plan proposes additions, changes, replacements, or destruction.

## Cost and cleanup

The **NAT gateway, public IPv4 address, ALB, EC2 instance and storage, RDS instance and storage, and managed secret** can incur charges while deployed, including between lab sessions. When finished, run `terraform plan -destroy`, review every resource, then run `terraform destroy` from **this exact project folder**. The database has `skip_final_snapshot = true` and `backup_retention_period = 0`, so cleanup will remove its test data without a final snapshot. Verify resource deletion in the AWS Console afterward.

**Do not delete, rename, or edit `terraform.tfstate` while the resources remain deployed.** It tracks what Terraform manages. Do not publish the state file or any database password. The account used for the first lab sign-in was the AWS root account; use a separate administrator identity for subsequent routine work when one is available.

## Current limitations

- The public ALB serves HTTP only. Do not enter real recipient, delivery, or payment information.
- The form and its deployment are unfinished; the health page alone does not accept orders.
- The database table was created manually; a new database deployment would need a schema migration.
- The EC2 role currently reads the RDS master secret for the lab. A production app should use a narrower database user and permissions.
- The lab has one app instance and one NAT gateway; it is not an availability or scale test.
