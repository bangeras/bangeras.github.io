---
title: iac-terraform-aws-demo
date: 2024-01-10 10:48:30
categories: [IAC, Terraform]
tags: [cloud, iac, aws]     # TAG names should always be lowercase
---

[![Source Code](/assets/img/sourcecode.png "Source code"){: width="30" height="35" }](https://github.com/bangeras/iac-terraform-aws-demo){:target="_blank"}{:rel="noopener noreferrer"}

## Overview
Terraform(IaC) framework to demonstrate some capabilitie for fundamental AWS design patterns:

### VPC and AZ setup
![VPC Architecture](/assets/img/iac-terraform-aws-demo/vpc_ec2ecs_architecture.png){: width="500" height="500" }

### APIGateway -> Lambda setup
```
# Lambda function
resource "aws_lambda_function" "payment_api_lambda_function" {
  function_name = "payment_api-lambda-function"
  runtime       = "python3.8"
  handler       = "payment_lambda_function.lambda_handler"
  filename      = "./lambda_functions/function.zip"
  role          = aws_iam_role.payment_lambda_function_execution_role.arn
}

# API Gateway
resource "aws_api_gateway_rest_api" "payment_api" {
  name        = "${var.vpc_suffix}-payment_api"
  description = "Payment API Gateway integrated with Lambda functions"

  endpoint_configuration {
    types = ["REGIONAL"]
  }
}

# API Gateway Resource
resource "aws_api_gateway_resource" "payment_api_resource" {
  rest_api_id = aws_api_gateway_rest_api.payment_api.id
  parent_id   = aws_api_gateway_rest_api.payment_api.root_resource_id
  path_part   = "payment"
}

# API Gateway Method
resource "aws_api_gateway_method" "payment_api_resource_method" {
  rest_api_id   = aws_api_gateway_rest_api.payment_api.id
  resource_id   = aws_api_gateway_resource.payment_api_resource.id
  http_method   = "ANY"
  authorization = "NONE"
}

# Lambda Integration with API Gateway
resource "aws_api_gateway_integration" "payment_api_integration" {
  rest_api_id             = aws_api_gateway_rest_api.payment_api.id
  resource_id             = aws_api_gateway_resource.payment_api_resource.id
  http_method             = aws_api_gateway_method.payment_api_resource_method.http_method
  integration_http_method = "POST" # Set your desired HTTP method
  # The type of integration with the specified backend. The valid value is
  #   http or http_proxy: for integration with an HTTP backend
  #   aws_proxy: for integration with AWS Lambda functions;
  #   aws: for integration with AWS Lambda functions or other AWS services, such as Amazon DynamoDB, Amazon Simple Notification Service or Amazon Simple Queue Service;
  #   mock: for integration with API Gateway without invoking any backend.

  type = "AWS_PROXY"
  uri  = aws_lambda_function.payment_api_lambda_function.invoke_arn
}

# API Gateway Deployment
resource "aws_api_gateway_deployment" "payment_api_deployment" {
  depends_on = [aws_api_gateway_integration.payment_api_integration]

  rest_api_id = aws_api_gateway_rest_api.payment_api.id
  stage_name  = "dev" # Set your desired stage name
}

# Used to give an external source (like an API Gateway, EventBridge Rule, SNS, or S3) permission to access the Lambda function
resource "aws_lambda_permission" "apigw_lambda" {
  statement_id  = "AllowExecutionFromAPIGateway"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.payment_api_lambda_function.function_name
  principal     = "apigateway.amazonaws.com"

  # More: https://repost.aws/questions/QUizsg_qznQLWKtqUD8Ruszw/api-gateway-lacks-permissions-to-trigger-lambda-when-made-by-terraform
  source_arn = "${aws_api_gateway_rest_api.payment_api.execution_arn}/*/*/*"
}


# API Gateway Resource
resource "aws_api_gateway_resource" "payment_api_notification_resource" {
  rest_api_id = aws_api_gateway_rest_api.payment_api.id
  parent_id   = aws_api_gateway_rest_api.payment_api.root_resource_id
  path_part   = "payment_notification"
}

# API Gateway Method
resource "aws_api_gateway_method" "payment_api_notification_resource_method" {
  rest_api_id   = aws_api_gateway_rest_api.payment_api.id
  resource_id   = aws_api_gateway_resource.payment_api_notification_resource.id
  http_method   = "ANY"
  authorization = "NONE"
}

# Lambda Integration with API Gateway
resource "aws_api_gateway_integration" "payment_api_notification_integration" {
  rest_api_id             = aws_api_gateway_rest_api.payment_api.id
  resource_id             = aws_api_gateway_resource.payment_api_notification_resource.id
  http_method             = aws_api_gateway_method.payment_api_notification_resource_method.http_method
  integration_http_method = "POST" # Set your desired HTTP method
  # The type of integration with the specified backend. The valid value is
  #   http or http_proxy: for integration with an HTTP backend
  #   aws_proxy: for integration with AWS Lambda functions;
  #   aws: for integration with AWS Lambda functions or other AWS services, such as Amazon DynamoDB, Amazon Simple Notification Service or Amazon Simple Queue Service;
  #   mock: for integration with API Gateway without invoking any backend.

  type = "AWS"
  uri  = aws_lambda_function.payment_api_lambda_function.invoke_arn
}
``` 
{: file="compute.tf" }

### ALB -> EC2 / ECS Fargate setup
```
resource "aws_launch_template" "ec2_launch_template" {
  name_prefix            = "${var.vpc_suffix}-ec2-launch_template"
  image_id               = "ami-00952f27cf14db9cd"
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]
  user_data              = filebase64("ec2-user-data.sh")
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "${var.vpc_suffix}-asg-ec2-paymentservice"
    }
  }
}

resource "aws_autoscaling_group" "asg" {
  name = "${var.vpc_suffix}-asg"
  #  availability_zones   = local.availability_zones
  desired_capacity     = 2
  max_size             = 4
  min_size             = 1
  health_check_type    = "EC2"
  termination_policies = ["OldestInstance"]
  vpc_zone_identifier  = aws_subnet.private_subnet.*.id
  target_group_arns    = [aws_lb_target_group.ec2_tg.arn]

  launch_template {
    id      = aws_launch_template.ec2_launch_template.id
    version = "$Latest"
  }
}

# Create a new load balancer attachment
resource "aws_autoscaling_attachment" "asg_alb" {
  autoscaling_group_name = aws_autoscaling_group.asg.id
  lb_target_group_arn    = aws_lb_target_group.ec2_tg.arn
}
```
{: file="compute-ec2-asg.tf"}

```
resource "aws_alb" "ecs_services_alb" {
  name            = "${var.vpc_suffix}-ecs-paymentservice-alb"
  subnets         = aws_subnet.public_subnet.*.id
  security_groups = [aws_security_group.alb_sg.id]
}

resource "aws_alb_target_group" "ecs_tg" {
  name        = "ecs-paymentservice-tg"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = aws_vpc.vpc.id
  target_type = "ip" # for Fargate

  #  health_check {
  #    healthy_threshold   = "3"
  #    interval            = "30"
  #    protocol            = "HTTP"
  #    matcher             = "200"
  #    timeout             = "3"
  #    path                = var.health_check_path
  #    unhealthy_threshold = "2"
  #  }
}

# aws_lb_target_group_attachment. That is primarily for attaching EC2 instances to a target group if you have no auto-scaling in place.
#For auto-scaled EC2 instance, or ECS managed containers, you let those service manage the target group attachment for you.


# Redirect all traffic from the ALB to the target group
resource "aws_alb_listener" "ec2_services_alb_listener" {
  load_balancer_arn = aws_alb.ecs_services_alb.id
  port              = 80
  protocol          = "HTTP"

  default_action {
    target_group_arn = aws_alb_target_group.ecs_tg.arn
    type             = "forward"
  }
}

# ECR repo
resource "aws_ecr_repository" "main" {
  name                 = "${var.vpc_suffix}-ecr"
  image_tag_mutability = "MUTABLE"
}

resource "aws_ecr_lifecycle_policy" "main" {
  repository = aws_ecr_repository.main.name

  policy = jsonencode({
    rules = [{
      rulePriority = 1
      description  = "keep last 10 images"
      action = {
        type = "expire"
      }
      selection = {
        tagStatus   = "any"
        countType   = "imageCountMoreThan"
        countNumber = 10
      }
    }]
  })
}

# ECS Cluster
resource "aws_ecs_cluster" "fargate_cluster" {
  name = "${var.vpc_suffix}-ecs-cluster"
}

# ECS Task definition
resource "aws_ecs_task_definition" "fargate_task" {
  family                   = "paymentservice-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]

  cpu           = "256"                                    # Set CPU units for the task
  memory        = "512"                                    # Set memory for the task
  task_role_arn = aws_iam_role.ecs_task_execution_role.arn # Specify your task execution role ARN

  container_definitions = jsonencode([{
    name  = "${var.vpc_suffix}-container-paymentservice"
    image = "nginx:latest"
    portMappings = [{
      containerPort = 80
      hostPort      = 80
    }]
  }])
}

resource "aws_ecs_service" "main" {
  name                               = "${var.vpc_suffix}-service-${var.env}"
  cluster                            = aws_ecs_cluster.fargate_cluster.id
  task_definition                    = aws_ecs_task_definition.fargate_task.arn
  desired_count                      = 3
  deployment_minimum_healthy_percent = 50
  deployment_maximum_percent         = 200
  launch_type                        = "FARGATE"

  network_configuration {
    security_groups  = [aws_security_group.ecs_sg.id]
    subnets          = aws_subnet.private_subnet.*.id
    assign_public_ip = false
  }

  load_balancer { # For ECS this configuration is used instead of aws_lb_target_group_attachment
    target_group_arn = aws_alb_target_group.ecs_tg.arn
    container_name   = "${var.vpc_suffix}-container-paymentservice"
    container_port   = 80
  }

  lifecycle {
    ignore_changes = [task_definition, desired_count]
  }
}
```
{: file="compute-ecs-fargate.tf"}

## Commands
```shell
    $ terraform init

    $ terraform validate

    $ terraform fmt

    $ terraform plan -var-file="./dev/terraform.tfvars"

    $ terraform apply -var-file="./dev/terraform.tfvars"

    $ terraform destroy -var-file="./dev/terraform.tfvars"
```