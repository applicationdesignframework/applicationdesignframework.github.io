# High-level application design from use case to code
**Note:** It is not necessarily the actual Amazon EC2 design.

## Use case
Company wants to build an Amazon EC2 product. The first use case is to allow customers launch instances.

### Business flow
![](https://github.com/alexpulver/adf/assets/4362270/e39e7d4c-aea3-4130-b1b8-60da4187745c)

## Features and stories
Feature: Manage EC2 instances
* User story: As a developer, I want to launch an instance

## Requirements
Requirements are not described in this example for brevity.

## Architecture

### Technical flow
![](https://github.com/alexpulver/adf/assets/4362270/2108fb96-a317-4498-8db5-8ba7005fc17a)

### Application boundaries
**Context**

We need to decide on application boundaires based on the technical flow.

**Decision**

The decision is to create two applications: _EC2 Instances Console_ and _EC2 Instances Control Plane_. 

**Consequences**

### EC2 Instances Console components

**Context**

We need to identify EC2 Instances Console components by describing requirements on technical level.

**Decision**

_EC2 Instances Console_ application contains _Toolchain_ and _Service_ components. _Toolchain_ component contains _Deployment Pipeline_ and _Pull Request Build_ components. _Service_ component contains _Network_, _Ingress_, _Compute_, _WebApp_, _Database_ and _Monitoring_ components. _Toolchain_ and _Service_ resources deploy as a stack each.

![image](https://github.com/alexpulver/adf/assets/4362270/faa17dc2-ed88-4e8d-b721-b6f421593a11)

**Consequences**

### EC2 Instances Control Plane components

**Context**

We need to identify EC2 Instances Control Plane components by describing requirements on technical level.

**Decision**

_EC2 Instances Control Plane_ is not described in this example for brevity.

**Consequences**

## Code

### Git repositories
EC2 Instances Console: `ec2-instances-console`

### Project structure

**EC2 Instances Console**

The example uses AWS Cloud Development Kit (AWS CDK) pseudo code for resources configuration. It should be possible to use the same approach with other cloud automation tools.

```
service/
    ui/
        Dockerfile
        app.py
        instances.py
    compute.py
        class Compute(Construct):
            ec2.SecurityGroup
            ecs.Cluster
            ecs.Service
            ecs.TaskDefinition
            ecr_assets.DockerImageAsset
    database.py
        class Database(Construct):
            ec2.SecurityGroup
            elasticache.CacheCluster
    ingress.py
        class Ingress(Construct):
            ec2.SecurityGroup
            elasticloadbalancingv2.NetworkLoadBalancer
            elasticloadbalancingv2.TargetGroup
            certificatemanager.Certificate
            route53.HostedZone
            route53.CNAMERecord
            waf.WebACL
    monitoring.py
        class Monitoring(Construct):
            cloudwatch.Metric
            cloudwatch.Alarm
            cloudwatch.Dashboard
            xray.Group
    network.py
        class Network:
            ec2.VPC
            ec2.Subnet
            ec2.RouteTable
    service_stack.py
        class ServiceStack(Stack):
            compute.Compute
            database.Database
            ingress.Ingress
            monitoring.Monitoring
            network.Network
toolchain/
    deployment_pipeline.py
        class DeploymentPipeline(Construct):
            pipelines.CodePipeline
    pull_request_build.py
        class PullRequestBuild(Construct):
            codebuild.Project
    toolchain_stack.py
        class ToolchainStack(Stack):
            toolchain.deployment_pipeline.DeploymentPipeline
            toolchain.pull_request_build.PullRequestBuild
app.py
    service.service_stack.ServiceStack("EC2InstancesConsole-Service-Sandbox")
    toolchain.toolchain_stack.ToolchainStack("EC2InstancesConsole-Toolchain-Sandbox")
    toolchain.toolchain_stack.ToolchainStack("EC2InstancesConsole-Toolchain-Management")
```