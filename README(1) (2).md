# Basketry Terraform Lab — Progress README

**Status:** Functional lab deployed. The browser order form saves synthetic orders in RDS MySQL, and an Easter basket image stored in private S3 is served through the app server and load balancer. Terraform's final plan reported **No changes**.

**AWS Region:** `us-east-2` (Ohio)  
**Local project folder:** `D:\1Terraform\Main.tf\DBasketry-Terraform-Fresh`  
**Terraform configuration:** `Main.tf`, `app_deploy.tf`, `schema_deploy.tf`, `images.tf`, `images_access.tf`, `images_object.tf`, and `image_publish.tf`; application and asset files: `order.php`, `schema.sql`, `migrate.php`, and `easter-basket.png`  
**Last verified:** September 25, 2026

This is a hands-on learning environment for the Basketry local gift-ordering concept. It serves a health-check page, a test order form, and a product image over HTTP. Use made-up test details only; it is not ready to collect real customer orders.

## Architecture

```text
Browser → public Application Load Balancer → private EC2 (Apache/PHP)
                                                  │
                                                  └→ private RDS MySQL

Private EC2 → NAT gateway → outbound AWS services and package repositories
Private EC2 → reads image from private S3 bucket → serves image through ALB
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
| 14. SQL test | Initially created the `orders` table manually in MySQL, inserted one made-up `Happy Birthday` test order, and selected it. | The query returned one saved test row. This step was later automated in step 19. |
| 15. Secret access | Added a Terraform IAM role policy granting the EC2 role `secretsmanager:GetSecretValue` on this RDS secret. | `aws secretsmanager get-secret-value --query ARN` worked both as the Session Manager shell user and as the `apache` user, without displaying the password. |
| 16. Browser form | Added `/var/www/html/order.php` on EC2. The PHP form retrieves the managed RDS secret using the instance role and inserts synthetic orders with a prepared statement. | `php -l` found no syntax errors; an HTTP form submission reported success. A MySQL `SELECT` showed the submitted `Test Recipient 2` / `Thank You` row as ID 2. |
| 17. File recovery | Restored `Main.tf` using VS Code Timeline after an accidental edit. Refreshed the expired AWS CLI session and backed up the working file as `..\Basketry-Main-working-backup.txt`. | `terraform validate` succeeded and `terraform plan` reported **No changes** before further edits. The state file was preserved. |
| 18. Repeatable form installation | Added `order.php` and `app_deploy.tf` alongside `Main.tf`. Terraform added an SSM Command document and State Manager association to install the PHP file on the existing private EC2 instance. | Reviewed a plan for **2 to add, 0 to change, 0 to destroy**; apply added both resources. A fresh browser test order saved successfully. Final `terraform plan` reported **No changes**. |
| 19. Repeatable database schema | Added `schema.sql`, `migrate.php`, and `schema_deploy.tf`. Terraform added an SSM Command document and association that read the managed RDS secret through the EC2 role and ran `CREATE TABLE IF NOT EXISTS orders`. | Reviewed and applied **2 to add, 0 to change, 0 to destroy**. The browser saved a `Schema Test` order; a MySQL `SELECT` showed four rows, including earlier test orders. |
| 20. Private images bucket | Added `images.tf` with a unique-name S3 bucket, four public access block settings, and a bucket-name output. | Reviewed and applied **2 to add, 0 to change, 0 to destroy**. AWS CLI confirmed `BlockPublicAcls`, `IgnorePublicAcls`, `BlockPublicPolicy`, and `RestrictPublicBuckets` are all `true`. Bucket: `basketry-tf-images-fcfaf0b50d31d101013d6019dc`. |
| 21. App image permissions | Added `images_access.tf`, granting the EC2 app role `s3:ListBucket` on this bucket and `s3:GetObject` only under `images/`. | Reviewed and applied **1 to add, 0 to change, 0 to destroy**. `sudo -u apache` could list the empty prefix, and later `head-object` returned the image size and `image/png`. The role has no S3 upload or delete permission from this policy. |
| 22. Managed test image | Uploaded a chosen Easter basket PNG to `images/easter-basket.png`, copied it into the local project as `easter-basket.png`, and added `images_object.tf` so Terraform manages that S3 key. | Reviewed and applied **1 to add, 0 to change, 0 to destroy**. The image remained private in S3. Keep the local PNG alongside the Terraform files for future plans and applies. |
| 23. Publish image through app server | Added `image_publish.tf` with an SSM document and association that download the private S3 image, verify its hash, and install it in `/var/www/html/images/`. | Reviewed and applied **2 to add, 0 to change, 0 to destroy**. The browser displayed `/images/easter-basket.png` through the ALB. The final `terraform plan` reported **No changes**. |

## Current test page

The Terraform output `basketry_alb_dns_name` gives the ALB hostname. The home page currently displays:

> Basketry Terraform app server is running

The browser test form is at `http://<ALB DNS name>/order.php`. Synthetic orders were verified in MySQL. The test image is at `http://<ALB DNS name>/images/easter-basket.png`; it is served by Apache after the app role reads the private S3 object. The original `/` health page remains in place for load balancer health checks.

## Where to resume

1. Open the same project folder in VS Code. Keep all `.tf` files, `order.php`, `schema.sql`, `migrate.php`, `easter-basket.png`, and `terraform.tfstate` together. Run Terraform in the VS Code PowerShell terminal in that folder.
2. If AWS authentication has expired, run `aws login --profile default --region us-east-2`. Confirm the intended AWS account and region. Run `terraform plan`; the last verified result was **No changes**.
3. The functional infrastructure lab is complete. For a later, separate hardening exercise, add HTTPS with a domain and certificate, use a limited database user, and improve app deployment and availability before accepting real orders.
4. When finished using the AWS lab, review a destruction plan and clean up billable resources. The S3 test image is now tracked by Terraform, so it will be removed with the bucket during a successful destroy.

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

The **NAT gateway, public IPv4 address, ALB, EC2 instance and storage, RDS instance and storage, managed secret, S3 storage and requests** can incur charges while deployed, including between lab sessions. When finished, run `terraform plan -destroy`, review every resource, then run `terraform destroy` from **this exact project folder**. The database has `skip_final_snapshot = true` and `backup_retention_period = 0`, so cleanup will remove its test data without a final snapshot. Verify resource deletion in the AWS Console afterward. The image bucket uses `force_destroy = false`; if you add other objects manually, remove them before destroying the bucket.

**Do not delete, rename, or edit `terraform.tfstate` while the resources remain deployed.** It tracks what Terraform manages. Do not publish the state file or any database password. The account used for the first lab sign-in was the AWS root account; use a separate administrator identity for subsequent routine work when one is available.

## Current limitations

- The public ALB serves HTTP only. Do not enter real recipient, delivery, or payment information.
- The form is deployed through an SSM association, but its HTTP endpoint is only suitable for synthetic lab orders. The PHP code retrieves the RDS master credentials when a valid order is submitted; a production app should use a narrower database user.
- The SQL schema is stored in the project and applied by an SSM association. The test rows remain in RDS; a fresh database would get the empty table, not the existing rows.
- The EC2 role currently reads the RDS master secret for the lab. A production app should use a narrower database user and permissions.
- The lab has one app instance and one NAT gateway; it is not an availability or scale test.
