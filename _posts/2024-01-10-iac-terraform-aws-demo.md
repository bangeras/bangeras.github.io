---
title: iac-terraform-aws-demo
date: 2024-01-10 10:48:30
categories: [IAC, Terraform]
tags: [cloud, iac, aws]     # TAG names should always be lowercase
---

## Overview

Terraform(IaC) framework to demonstrate some capabilitie for fundamental AWS design patterns:
### VPC and AZ setup

### APIGateway -> Lambda setup

### ALB -> EC2 / ECS Fargate setup

## Commands
```shell
    $ terraform init

    $ terraform validate

    $ terraform fmt

    $ terraform plan -var-file="./dev/terraform.tfvars"

    $ terraform apply -var-file="./dev/terraform.tfvars"

    $ terraform destroy -var-file="./dev/terraform.tfvars"
```