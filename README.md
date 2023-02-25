# Prefect 2 Agent on ECS Fargate

This recipe demonstrates how to deploy a Prefect 2 agent onto ECS Fargate using [Terraform](https://www.terraform.io/). It is intended to be used as a Terraform module as described in [Usage](#usage) below. It assumes you have Terraform installed, and was tested with Terraform `v1.2.7`.

Note that flows will run inside the agent ECS task, as opposed to becoming their own ECS tasks.

To start with you will need your Prefect account ID, workspace ID, and API key. You will also need to pick one or more subnets that Fargate will launch into, as well as give your deployment a name.

In order to avoid accidentally committing your API key, consider structuring your project as follows,

```
.
├── main.tf
├── terraform.tfvars
└── variables.tf
```

```hcl
// variables.tf
variable prefect_api_key {}
```

```hcl
// terraform.tfvars
prefect_api_key      = "pnu_******"
prefect_account_id   = "a22******"
prefect_workspace_id = "e4c******"
vpc_id               = "vpc-******"
agent_subnets        = ["subnet-******"]

```

Assuming the file structure above, you can run `terraform init` followed by `terraform apply` to create the resources. Check out the [Inputs](#inputs) section below for more options.

# Set up and run Prefect First Task

Flow & Deployment is based on [Prefect Tutorial](https://docs.prefect.io/tutorials/storage/)

```
# ECS component with Terraform
$ cd tfs
$ terraform init
$ terraform apply

# Prefect
$ source env/bin/activate
$ prefect deployment build ./log_flow.py:log_flow -n log-agent-ecs -sb s3/ecs-agent-bucket -q test-queue -o log-flow-agent-ecs.yaml
$ prefect deployment apply log-flow-agent-ecs.yaml --upload
$ prefect deployment run 'log-flow/log-agent-ecs'
```
