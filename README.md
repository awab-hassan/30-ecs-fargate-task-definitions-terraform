# ArcherHealth — ECS Fargate Task Definitions (stage)

Terraform fragment defining the two **ECS Fargate task definitions** for the ArcherHealth staging stack: a lightweight **backend** task (`512 MB / 0.25 vCPU`, port 8000) and a heavier **frontend** task (`2048 MB / 1 vCPU`, port 3000). Both share the same execution role and are parameterised on container image tags so CI can swap images without editing Terraform.

## Highlights

- **Two separately sized tasks** — backend stays cheap; frontend gets 4× the memory and 4× the CPU because SSR/Node builds are the hot path.
- **Image tags externalised** — `var.backend_image` / `var.frontend_image` get set from the CI pipeline (CodePipeline + ECR push).
- **awsvpc networking** — each task gets its own ENI, enabling security-group-per-service and ALB target-group attach.

## Tech stack

- Terraform + AWS provider
- AWS ECS Fargate, IAM execution role, ECR (images), ALB (upstream)

## Expected variables

```hcl
variable "backend_image"  {}   # ECR URI, e.g. <acct>.dkr.ecr.<region>.amazonaws.com/archerhealth-backend:<tag>
variable "frontend_image" {}   # same pattern for frontend
```

Plus an `aws_iam_role.ecs_task_execution_role` (standard `AmazonECSTaskExecutionRolePolicy`) referenced by `execution_role_arn`.

## Notes

- Staging-only shape; a prod equivalent would typically bump memory/cpu further and add a `task_role_arn` for app-level AWS calls.
- Demonstrates: Fargate sizing per workload, image-tag parameterisation for CI-driven deploys, awsvpc networking choice.
