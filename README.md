# Set up

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
// Don't panic! This isn't a real API key
prefect_api_key = "pnu_bcf655365883614d468990896264f6a30372"
```

```hcl
// main.tf

provider "aws" {
  region = "ap-northeast-1"
}

// Don't panic! These values are just random uuid.uuid4()s
module "prefect_ecs_agent" {
  source = "github.com/PrefectHQ/prefect-recipes//devops/infrastructure-as-code/aws/tf-prefect2-ecs-agent"

  agent_subnets        = [
    "subnet-014aa5f348034e45b",
    "subnet-df23ae9eab1f49af9"
  ]
  name                 = "dev"
  prefect_account_id   = "a2209e0b-749e-4caa-8767-9db40bdec243"
  prefect_api_key      = var.prefect_api_key
  prefect_workspace_id = "e4ce4a42-dffa-42af-abc7-39652df4b8e3"
  vpc_id               = "vpc-09687bc1b3a1f4bff"
}
```

Assuming the file structure above, you can run `terraform init` followed by `terraform apply` to create the resources. Check out the [Inputs](#inputs) section below for more options.

## Reference

The [terraform docs](https://terraform-docs.io/) below can be generated with the following command:

```sh
terraform-docs markdown table . --output-file README.md
```

<!-- BEGIN_TF_DOCS -->

## Requirements

| Name                                                   | Version |
| ------------------------------------------------------ | ------- |
| <a name="requirement_aws"></a> [aws](#requirement_aws) | ~> 4.0  |

## Providers

| Name                                             | Version |
| ------------------------------------------------ | ------- |
| <a name="provider_aws"></a> [aws](#provider_aws) | 4.27.0  |

## Modules

No modules.

## Resources

| Name                                                                                                                                                                                      | Type        |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| [aws_cloudwatch_log_group.prefect_agent_log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group)                                      | resource    |
| [aws_ecs_cluster.prefect_agent_cluster](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster)                                                          | resource    |
| [aws_ecs_cluster_capacity_providers.prefect_agent_cluster_capacity_providers](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster_capacity_providers) | resource    |
| [aws_ecs_service.prefect_agent_service](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_service)                                                          | resource    |
| [aws_ecs_task_definition.prefect_agent_task_definition](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_task_definition)                                  | resource    |
| [aws_iam_role.prefect_agent_execution_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)                                                         | resource    |
| [aws_iam_role.prefect_agent_task_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)                                                              | resource    |
| [aws_secretsmanager_secret.prefect_api_key](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret)                                            | resource    |
| [aws_secretsmanager_secret_version.prefect_api_key_version](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret_version)                    | resource    |
| [aws_security_group.prefect_agent](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)                                                            | resource    |
| [aws_security_group_rule.https_outbound](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)                                                 | resource    |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region)                                                                               | data source |

## Inputs

| Name                                                                                                               | Description                                                                                      | Type           | Default                            | Required |
| ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | -------------- | ---------------------------------- | :------: |
| <a name="input_agent_cpu"></a> [agent_cpu](#input_agent_cpu)                                                       | CPU units to allocate to the agent                                                               | `number`       | `1024`                             |    no    |
| <a name="input_agent_desired_count"></a> [agent_desired_count](#input_agent_desired_count)                         | Number of agents to run                                                                          | `number`       | `1`                                |    no    |
| <a name="input_agent_extra_pip_packages"></a> [agent_extra_pip_packages](#input_agent_extra_pip_packages)          | Packages to install on the agent assuming image is based on prefecthq/prefect                    | `string`       | `"prefect-aws s3fs"`               |    no    |
| <a name="input_agent_image"></a> [agent_image](#input_agent_image)                                                 | Container image for the agent. This could be the name of an image in a public repo or an ECR ARN | `string`       | `"prefecthq/prefect:2-python3.10"` |    no    |
| <a name="input_agent_log_retention_in_days"></a> [agent_log_retention_in_days](#input_agent_log_retention_in_days) | Number of days to retain agent logs for                                                          | `number`       | `30`                               |    no    |
| <a name="input_agent_memory"></a> [agent_memory](#input_agent_memory)                                              | Memory units to allocate to the agent                                                            | `number`       | `2048`                             |    no    |
| <a name="input_agent_queue_name"></a> [agent_queue_name](#input_agent_queue_name)                                  | Prefect queue that the agent should listen to                                                    | `string`       | `"default"`                        |    no    |
| <a name="input_agent_subnets"></a> [agent_subnets](#input_agent_subnets)                                           | Subnets to place the agent in                                                                    | `list(string)` | n/a                                |   yes    |
| <a name="input_agent_task_role_arn"></a> [agent_task_role_arn](#input_agent_task_role_arn)                         | Optional task role ARN to pass to the agent. If not defined, a task role will be created         | `string`       | `null`                             |    no    |
| <a name="input_name"></a> [name](#input_name)                                                                      | Unique name for this agent deployment                                                            | `string`       | n/a                                |   yes    |
| <a name="input_prefect_account_id"></a> [prefect_account_id](#input_prefect_account_id)                            | Prefect cloud account ID                                                                         | `string`       | n/a                                |   yes    |
| <a name="input_prefect_api_key"></a> [prefect_api_key](#input_prefect_api_key)                                     | Prefect cloud API key                                                                            | `string`       | n/a                                |   yes    |
| <a name="input_prefect_workspace_id"></a> [prefect_workspace_id](#input_prefect_workspace_id)                      | Prefect cloud workspace ID                                                                       | `string`       | n/a                                |   yes    |
| <a name="input_vpc_id"></a> [vpc_id](#input_vpc_id)                                                                | VPC ID in which to create all resources                                                          | `string`       | n/a                                |   yes    |

## Outputs

| Name                                                                                                                                | Description |
| ----------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| <a name="output_prefect_agent_cluster_name"></a> [prefect_agent_cluster_name](#output_prefect_agent_cluster_name)                   | n/a         |
| <a name="output_prefect_agent_execution_role_arn"></a> [prefect_agent_execution_role_arn](#output_prefect_agent_execution_role_arn) | n/a         |
| <a name="output_prefect_agent_security_group"></a> [prefect_agent_security_group](#output_prefect_agent_security_group)             | n/a         |
| <a name="output_prefect_agent_service_id"></a> [prefect_agent_service_id](#output_prefect_agent_service_id)                         | n/a         |
| <a name="output_prefect_agent_task_role_arn"></a> [prefect_agent_task_role_arn](#output_prefect_agent_task_role_arn)                | n/a         |

<!-- END_TF_DOCS -->
