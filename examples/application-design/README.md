# High-level application design from use case to code

This example focuses on establishing application boundary and mapping architecture to code. Requirements are out of scope. **Note:** It is not necessarily the actual Amazon EC2 design.

## Working backwards

Company wants to build Amazon EC2 product. The first use case is to allow customers launch instances.

![image](https://github.com/alexpulver/adf/assets/4362270/4513c22e-485b-45a3-be72-7d166d2cd275)

## Stories and flows

First story is "As a developer, I want to launch an instance". Flow describes developer experience launching an instance using AWS Management Console.

![image](https://github.com/alexpulver/adf/assets/4362270/e39e7d4c-aea3-4130-b1b8-60da4187745c)

## Architecture

### Stories and flows

Describing story and flow on technical level helps to establish application boundaries. The decision is to create two applications: _EC2 Instances Console_ and _EC2 Instances Control Plane_.

![image](https://github.com/alexpulver/adf/assets/4362270/2108fb96-a317-4498-8db5-8ba7005fc17a)

### Architectural decision records (ADRs)

_EC2 Instances Console_ application includes Toolchain and Service components. Toolchain includes Deployment Pipeline and Pull Request Build components. Service includes Network, Ingress, Compute, WebApp, Database and Monitoring components. Components use infrastructure services and runtime code to provide functionality. Application (metadata), Toolchain and Service resources deploy as a stack each.

![image](https://github.com/alexpulver/adf/assets/4362270/faa17dc2-ed88-4e8d-b721-b6f421593a11)

_EC2 Instances Control Plane_ is out of scope.

### Code structure

_EC2 Instances Console_

The example uses AWS Cloud Development Kit (AWS CDK) pseudo code for infrastructure services. It should be possible to use the same approach with other infrastructure automation tools.

```
service/
    webapp/
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