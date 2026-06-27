# Day 10 — AWS CLI ☁️

## LinkedIn Post:

```
☁️ [Day 10/15] DevOps Cheat Sheet: AWS CLI

Console clicking is for demos.
Real DevOps engineers use the CLI.

Here are AWS CLI one-liners I use weekly:

━━━━━━━━━━━━━━━━━

# "Who am I?"
aws sts get-caller-identity

# EC2 — list all instances (clean table)
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PrivateIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# S3 — the Swiss army knife
aws s3 ls s3://my-bucket --recursive --human-readable
aws s3 sync ./local-dir s3://my-bucket/prefix/
aws s3 presign s3://my-bucket/file.txt --expires-in 3600  # Temp download link

# CloudWatch Logs — tail like a local file
aws logs tail /aws/lambda/my-function --follow
aws logs filter-log-events \
  --log-group-name /aws/ecs/app \
  --filter-pattern "ERROR"

# RDS — quick status check
aws rds describe-db-instances \
  --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceStatus,Engine]' \
  --output table

# Quick snapshot before risky changes
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snap-$(date +%Y%m%d)

━━━━━━━━━━━━━━━━━

💡 Pro tip: `--query` with JMESPath is a game changer.

Instead of piping through jq, use --query to filter
AWS responses directly. Cleaner, faster, scriptable.

Also: `aws s3 presign` — generate temporary download
links without making buckets public. Security win. 🔒

♻️ Repost for your AWS community
#AWS #CloudEngineering #DevOps #CLI #SRE #Cloud
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Covers EC2, S3, IAM, RDS, CloudWatch & more!

Tomorrow: Terraform — state commands that saved my infrastructure 🏗️
```
