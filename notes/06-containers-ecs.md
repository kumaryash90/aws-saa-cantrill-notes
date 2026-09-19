# Containers and ECS

## Intro

- Containerization solves virtualization overhead problems

***todo: diagram***

- docker image and containers
    - fs layers are shared, only r/w are unique to each container, hence more efficient

***todo: diagram***

- [docker image on ec2 demo](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0030-aws-associate-ec2docker/ec2docker_AL2023.yaml&stackName=EC2DOCKER)

## ECS Concepts

- what ec2 is to virtual machines, ecs is to containers
- ecs two modes:
    - ec2 mode
    - fargate mode
- it's a managed service
- ecr (similar to dockerhub)
- overview
    - container definition — image and ports
    - task definition — security (task role), containers, resources, temp cred
    - you deploy tasks or services in an ecs cluster (not containers directly)
    - a task can have 1 or more containers
    - task roles — best practice, to give containers in ecs access to aws resources
    - service definition — because a task doesn't scale on its own, and isn't HA on its own
        - define how many task copies, how task scales, load balancing, etc

## ECS Cluster Modes

- ec2 mode
    - you're expected to manage all of this yourself, not serverless
    - it's a good middle ground, gives your flexibility/control

    ***todo: diagram***

- fargate mode
    - removes a lot of management overhead, no servers to manage

    ***todo: diagram***

- be able to pick b/w ec2 vs ecs (ec2) vs ecs (fargate), based on business case

## ECR

- it's like dockerhub, for aws — a managed container registry service
- each aws acc has public and private registry
- integrated with IAM

## Kubernetes 101

- (it's cloud agnostic)
- a system for automating deployment, scaling, and mgmt of containerized apps

***todo: diagram***

## EKS 101

- it's just AWS' implementation of kubernetes, fully managed
    - aws also released it as open-source as EKS Distro
    - *(Kubernetes as a Service)*
- control plane scales and runs on multiple AZs
- integrates with AWS services like ECR, ELB, IAM, VPC
- EKS cluster = EKS control plane + EKS nodes
- etcd distributed across multiple AZs
- Nodes — self-managed, managed node groups, or fargate pods

***todo: diagram***
