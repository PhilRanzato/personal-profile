---
title: "AWS Kinesis Lambda"
date: 2021-05-04T11:30:13+05:30
draft: false
img: /images/projects/aws-kinesis.webp
description: "CICD pipeline with Terraform and Github Actions to deliver a Lambda function on AWS"
github: https://github.com/PhilRanzato/lambda-aws
---

![Terraform Deploy](https://github.com/PhilRanzato/lambda-aws/actions/workflows/cicd.yml/badge.svg)

This repository is the starting point for an Infrastructure as Code pipeline.

## AWS Lambda Function

The repository contains a AWS Lambda Function written in Python3 that processes AWS Kinesis Data Stream logs and writes the processed results into a AWS DynamoDB table.

Example of input **before** processing:

```json
{
    "id": 33,
    "message": "This function will count the number of occurrences of the word error."
}
```

Example of input **after** processing:

```json
{
    "id": 33,
    "message": "This function will count the number of occurrences of the word error.",
    "errors": 1
}
```

## Continuous Integration and Continuous Deployment

Each time a new Pull Request is created (and/or a new push on master), the IaC pipeline (here implemented as GitHub action) starts and deploys the whole infrastructure using Terraform:

- An AWS Kinesis Data Stream named `application-logs-stream`
- An AWS DynamoDB table named `application-logs-table`
- An AWS Lambda function named `kinesis-to-dynamodb`

The pipeline takes care of:

- Job **`Lint`**: Linting Lambda Python3 code.
- Job **`Terraform-deploy`**: Zipping the file [`lambda.py`](https://github.com/PhilRanzato/lambda-aws/blob/master/lambda.py) into a `function.zip` that will be moved to the Terraform repository and uploaded as AWS Lambda source file. The result file will also be pushed on a new branch starting from the HEAD (it will be then deleted only for Pull Requests).
- Init Terraform backend on a AWS s3 bucket.
- Applying Terraform resources in two steps:
  1. Firstly the AWS Kinesis Data Stream and the AWS DynamoDB table.
  2. Then the Lambda function.
- Job **`Kinesis-test`**: Test the whole workflow by installing and configuring the AWS Kinesis Agent on the GitHub runner, using *repository secrets* divided into a *repository environment*. Then it runs a test Python application that produces 3 log lines that will be provided to the AWS Kinesis Data Stream using the AWS Kinesis Agent.
- Job **`Terraform-destroy`**: eventually, the pipeline will clean up the infrastructure by destroying all Terraform generated resources and, in the case of a Pull Request, remove the branch that contains the `function.zip` since it will be created again once merged.

> Terraform destroy should not be executed when the pipeline refers to master/release but for convenience, it is destroyed all the times.

Additionally, a Jenkinsfile example to deploy the workflow is provided in the repository.

To see more about the Terraform code, visit the below repository.

## References

- Python Application that produces logs -> [kinesis-application](https://github.com/PhilRanzato/kinesis-application)
- Terraform workflow -> [terraform-lambda-aws](https://github.com/PhilRanzato/terraform-lambda-aws)
