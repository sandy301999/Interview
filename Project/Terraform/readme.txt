# Terraform AWS Infrastructure Setup - Stepwise Guide

## 1. Bootstrap Remote State
- Create S3 bucket for state + enable versioning:
  aws s3api create-bucket --bucket my-tf-state-prod --create-bucket-configuration LocationConstraint=ap-south-1
  aws s3api put-bucket-versioning --bucket my-tf-state-prod --versioning-configuration Status=Enabled

- Create DynamoDB lock table:
  aws dynamodb create-table --table-name tf-state-locks \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST

## 2. Repo Layout
infra/
├─ modules/
│  ├─ vpc/
│  └─ ec2/
├─ envs/
│  ├─ prod/
│  └─ dev/
└─ globals/

## 3. Backend Configuration
- Each env has `backend.tf`:
  terraform {
    backend "s3" {
      bucket         = "my-tf-state-prod"
      key            = "envs/prod/terraform.tfstate"
      region         = "ap-south-1"
      dynamodb_table = "tf-state-locks"
      encrypt        = true
    }
  }

## 4. Providers & Tags
- Configure provider with default tags:
  provider "aws" {
    region = "ap-south-1"
    default_tags { tags = local.default_tags }
  }

- locals.tf:
  locals {
    default_tags = {
      Owner       = "sre"
      Environment = terraform.workspace
      ManagedBy   = "terraform"
      Project     = "core-platform"
    }
  }

## 5. Handle Secrets
- Store secrets in AWS SSM Parameter Store or Secrets Manager.
- Use data source:
  data "aws_ssm_parameter" "db_password" {
    name           = "/prod/db/password"
    with_decryption = true
  }

## 6. Workflow Commands
- Format & validate:
  terraform fmt -check
  terraform validate

- Init backend:
  terraform init

- Plan & apply:
  terraform plan -out plan.tfplan -var-file=envs/prod/prod.tfvars
  terraform apply plan.tfplan

- Import existing resource:
  terraform import aws_s3_bucket.logs org-logs

- Drift detection:
  terraform plan -refresh-only -detailed-exitcode -var-file=envs/prod/prod.tfvars

- Pull state for backup:
  terraform state pull > state.json

## 7. Jenkins Automation
- Jenkinsfile stages:
  1. Checkout
  2. terraform fmt / init / validate
  3. terraform plan -out plan.tfplan
  4. Manual approval
  5. terraform apply plan.tfplan

## 8. Monitoring with CloudWatch
- Example alarm:
  resource "aws_cloudwatch_metric_alarm" "cpu_high" {
    alarm_name          = "asg-cpu-high"
    comparison_operator = "GreaterThanThreshold"
    evaluation_periods  = 5
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 60
    statistic           = "Average"
    threshold           = 70
    alarm_actions       = [aws_sns_topic.alerts.arn]
    ok_actions          = [aws_sns_topic.alerts.arn]
  }

## 9. Best Practices
- Always use plan -> apply.
- Store secrets in SSM/Secrets Manager, never in tfvars.
- Tag everything with defaults + module tags.
- Use `create_before_destroy` lifecycle for downtime-sensitive resources.
- Enable drift detection in CI/CD.
- Backup state files using S3 versioning.

