# High-level application design from use case to code
**Note:** It is not necessarily the actual Amazon EC2 design.

## Use case
Offer virtual machines (instances) as a service.

### Diagram
![](/images/ec2-use-case.svg)

### Business flows
#### Launch instance
**Steps**
1. Navigate to Console Home and sign-in.
2. Navigate to EC2 Home Console.
3. Select the Instances menu.
4. Click on Launch, fill the form, and submit.

**Requirements**

Requirements are not described in this example for brevity.

## Architecture

### Diagram
![](/images/ec2-system-architecture.svg)

### Technical flows

#### Launch instance
**Steps**

Steps are not described in this example for brevity.

**Requirements**

Requirements are not described in this example for brevity.

### Architectural decision records (ADRs)

#### Application boundaries
**Context**

We need to define application boundaries.

**Decision**

The decision is to create two applications: _EC2 Instances Console_ and _EC2 Instances Control Plane_. 

**Consequences**

#### EC2 Instances Console components

**Context**

We need to define EC2 Instances Console components.

**Decision**

_EC2 Instances Console_ application contains _Toolchain_ and _Service_ components. _Toolchain_ component contains _Deployment Pipeline_ and _Pull Request Build_ components. _Service_ component contains _Network_, _Ingress_, _UI_, _Database_ and _Monitoring_ components. _Toolchain_ and _Service_ resources deploy as a stack each.

![](/images/ec2-application-architecture.svg)

**Consequences**

#### EC2 Instances Console technologies

**Context**

We need to choose technologies to implement EC2 Instances Console components.

**Decision**

![](/images/ec2-application-technology.svg)

**Consequences**


#### EC2 Instances Control Plane components

**Context**

We need to define EC2 Instances Control Plane components based on the technical flow.

**Decision**

_EC2 Instances Control Plane_ is not described in this example for brevity.

**Consequences**

## Code

### Git repositories
EC2 Instances Console: `ec2-instances-console`

### Project structure

**EC2 Instances Console**

The example uses AWS Cloud Development Kit (AWS CDK) pseudo code for resources configuration. It should be possible to use the same approach with other tools.

```
service/
    ui/
        app/
            Dockerfile
            instances.py
            main.py
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
            network.Network
            ingress.Ingress
            database.Database
            ui.compute.Compute
            monitoring.Monitoring
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
    toolchain.toolchain_stack.ToolchainStack("EC2InstancesConsole-Toolchain-Production")
```