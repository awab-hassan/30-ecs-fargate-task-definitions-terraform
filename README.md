# Project # 30 - ecs-fargate-task-definitions-terraform

Terraform module defining two ECS Fargate task definitions for a staging environment: a lightweight backend task and a heavier frontend task. Container image tags are passed as variables so CI can update images without editing Terraform.

This module covers task definitions only. The ECS cluster, service, ALB, target groups, and IAM execution role are expected to exist already and are referenced by the task definitions.

## Task Definitions

| Task | Memory | CPU | Container Port |
|---|---|---|---|
| backend | 512 MB | 0.25 vCPU | 8000 |
| frontend | 2048 MB | 1 vCPU | 3000 |

Both tasks use the `awsvpc` network mode, giving each task its own ENI and enabling per-service security groups and direct ALB target group attachment.

## Inputs

| Variable | Purpose |
|---|---|
| `backend_image` | Backend container image, e.g. `<acct>.dkr.ecr.<region>.amazonaws.com/<repo>:<tag>` |
| `frontend_image` | Frontend container image (same URI pattern) |

The module references an existing `aws_iam_role.ecs_task_execution_role` with the AWS-managed `AmazonECSTaskExecutionRolePolicy` attached.

## Stack

Terraform · AWS ECS Fargate · ECR · IAM · awsvpc networking

## Deployment

CI typically supplies image URIs from the most recent ECR push:

```bash
terraform init
terraform plan \
  -var="backend_image=<acct>.dkr.ecr.<region>.amazonaws.com/backend:<tag>" \
  -var="frontend_image=<acct>.dkr.ecr.<region>.amazonaws.com/frontend:<tag>"
terraform apply
```

## Notes

- **This module defines task definitions only.** It does not create the ECS cluster, service, ALB, target groups, or the execution IAM role. Those are provisioned separately and referenced as data sources or pre-existing resources.
- **No `task_role_arn`.** These tasks have no app-level AWS API permissions. Add a `task_role_arn` if the application needs to call AWS services directly (e.g. S3, DynamoDB, Secrets Manager).
- **Sizing is staging-tier.** A production equivalent should be tested against expected load and adjusted. Fargate billing is per-second based on the configured CPU and memory.
- **No log driver shown.** Verify the container definitions configure `logConfiguration` with `awslogs` to stream container output to CloudWatch.
