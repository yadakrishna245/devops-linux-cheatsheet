# Day 11 — Terraform 🏗️

## LinkedIn Post:

```
🏗️ [Day 11/15] DevOps Cheat Sheet: Terraform

"Who changed the infra?"
"We can't reproduce the staging environment."
"Someone deleted the security group manually."

Terraform fixes all three. Here's what I use daily:

━━━━━━━━━━━━━━━━━

# The workflow
terraform init                              # Download providers
terraform plan -out=tfplan                  # Preview changes
terraform apply tfplan                      # Apply safely
terraform destroy                           # Tear down

# State management (the dangerous but essential commands)
terraform state list                        # What's managed?
terraform state show aws_instance.web       # Details of a resource
terraform state mv aws_instance.old aws_instance.new  # Rename
terraform state rm aws_instance.deprecated  # Unmanage a resource
terraform import aws_instance.web i-123456  # Adopt existing infra

# Housekeeping
terraform fmt -recursive                    # Format all files
terraform validate                          # Syntax check

━━━━━━━━━━━━━━━━━

# Quick EC2 example:
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  subnet_id     = aws_subnet.private.id

  vpc_security_group_ids = [aws_security_group.web.id]

  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}

━━━━━━━━━━━━━━━━━

💡 Pro tip: ALWAYS use `terraform plan -out=tfplan`

Never apply without reviewing the plan first.
And never run `terraform apply` without -out flag in production.

One wrong `destroy` can wipe your entire infrastructure.
Ask me how I know. 💀

♻️ Repost for your IaC community
#Terraform #IaC #DevOps #AWS #CloudEngineering #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Includes S3 bucket, VPC, and more Terraform patterns!

Tomorrow: Ansible — managing 4000 servers with one playbook 📦
```
