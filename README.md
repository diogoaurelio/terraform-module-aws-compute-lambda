Terraform AWS module for Lambda function
========================================

![Image of Terraform](https://i.imgur.com/Jj2T26b.jpg)


# Intro

Flexible but yet opinionated AWS Lambda function module that creates:
- AWS Lambda function (see next to configure optional parameters) with opinionated tagging, optionally inside a specific VPC and security group;
- Zipping of a given directory to push on changes into AWS S3 bucket (externally injected);
- IAM role for running lambda function and base IAM policy;
- Externally provided KMS key ARN to encrypt lambda environment variable
- Cloudwatch group for storing Lambda logs

Optional:

- launch (or not) the Lambda function inside a VPC, by providing it a list of subnet(s) and another list of security group(s).
- provide it with a Dead Letter Queue (DLQ) configuration in order to handle failures, by providing either SNS or SQS ARN.
- provide additional IAM policies to extend Lambda permissions

# Usage

Example usage:

```hcl

module "main_lambda" {
  source                      = "github.com/diogoaurelio/terraform-module-aws-compute-lambda"
  version                     = "v0.0.1"

  aws_region                  = "eu-west-1"
  aws_account_id              = "012345678912"
  environment                 = "dev"
  project                     = "lambda"

  lambda_unique_function_name = "unique-lambda-name"
  runtime                     = "python3.6"
  handler                     = "handler"
  lambda_iam_role_name        = "unique-lambda-role"
  logs_kms_key_arn            = "<some-kms-key-arn>"

  main_lambda_file            = "main"
  lambda_source_dir           = "${local.main_lambda_source_dir}"

  lambda_zip_file_location    = "${path.cwd}/../../../etl/lambdas/news/main.zip"
  lambda_env_vars             = "${local.main_lambda_env_vars}"

  additional_policy           = "${data.aws_iam_policy_document.main_lambda_policy.json}"
  attach_policy               = true

  # configure Lambda function inside a specific VPC
  security_group_ids          = ["sg-012345678"]
  subnet_ids                  = ["subnet-12345678"]

  # DLQ
  use_dead_letter_config_target_arn = true
  dead_letter_config_target_arn     = "${aws_sns_topic.lambda_sns_dql.arn}"

}

# Locals used to specify lambda ENVIRONMENT variables
locals {

  lambda_env_vars = {
    ENVIRONMENT = "${var.environment}"
    REGION      = "${var.aws_region}"
  }
}

# optional additional policy document
data "aws_iam_policy_document" "additional_lambda_policy" {
  statement {
    effect = "Allow"

    actions = [
      "s3:GetBucketLocation",
      "s3:ListAllMyBuckets",
    ]

    resources = [
      "*",
    ]
  }
}

## SNS topic for lambda failures - Dead Letter Queue (DLQ)
resource "aws_sns_topic" "lambda_sns_dql" {
  name = "lambda-dlq-sns-topic"
}


```

## Local terraform setup

* [Install Terraform](https://www.terraform.io/)

```bash
brew install terraform
```

* In order to automatic format terraform code (and have it cleaner), we use pre-commit hook. To [install pre-commit](https://pre-commit.com/#install).

After installing pre-commit, you can set it up:
```bash
pre-commit install
```


* Run [pre-commit install](https://pre-commit.com/#usage) to setup locally hook for terraform code cleanup.

```bash
pre-commit install
```


# Authors/Contributors

See the list of [contributors](https://github.com/diogoaurelio/terraform-module-aws-compute-lambda/graphs/contributors) who participated in this project.