# End-to-end design process and application boundary evolution

This example describes design phases in high-level and how application boundary changes as part of company growth. It doesn't go into details of stories, requirements, architecture and architectural decision records.

## Initial design

### End-to-end process

![Product engineering](https://github.com/alexpulver/adf/assets/4362270/cef75431-b95c-4468-99e0-d3bd86d0ce2b)

### Working backwards

AnyCompany product team wants to build a DocuStar product that allows enterprises to store, view, edit, and share documents. Product team identified two main personas within customers - end-users and application security engineers. Working backwards from these personas, product team identified 3 main use cases: 1/ documents storage (end-users) 2/ user management (application security engineers) 3/ audit trail (application security engineers).

### Stories

Product team captured stories and flows for each use case. 

### Requirements

Product and engineering teams worked together to define the functional and non-functional requirements to satisfy the stories.

### Architecture 

Engineering team captured flows for each story on architecture level (e.g. sign in, get upload link, upload document, log activity) to identify applications and their components. 

Engineering team decided to create the following applications: 1/ Soteria (documents storage) 2/ Aegis (user management) 3/ Athena (audit trail). The engineering team then identified components for each application and their interactions.

Note: Aegis and Athena details are left out for brevity through the rest of the document.

_Soteria_

* UI - Authenticates users and allows them to interact with the application
* Service - Generates protected links for upload and download
* Repository - Stores the documents
* Monitoring - Displays business and operational metrics

Soteria interacts with Aegis and Athena to sign in users and log user activity respectively.

### Architectural decision records

Engineering team captured the design in architectural decision records (ADRs).

_Soteria_

* UI infrastructure includes Route 53, CloudFront, S3, CloudWatch, X-Ray, and Service Catalog services. UI runtime includes React application.
* Service infrastructure includes Route 53, Certificate Manager, WAF, EC2, Elastic Load Balancing, ECS, DynamoDB, CloudWatch, X-Ray, and Service Catalog services. Service runtime includes Express application.
* Repository infrastructure includes Route 53, S3, CloudWatch, X-Ray, and Service Catalog services.
* Monitoring infrastructure includes Route 53, CloudWatch, X-Ray, and Service Catalog services.

### Code structure

_Soteria_

Engineering team decided to create a single code repository for the Soteria application. The implementation includes components (UI, service, repository, and monitoring), toolchain (deployment pipeline and pull request build), and metadata (resources and attributes). Components, toolchain, and metadata deploy as a stack each. Components and toolchain resources are associated with the metadata.

Engineering team decided to create the following environments: 1/ per-builder sandbox environment 2/ management environment (toolchain and metadata) 3/ alpha, beta, gamma, and production environments (components). Toolchain deploys components to the alpha, beta, gamma and production environments.

```
Repo: soteria.git

components/
  ui/
    runtime/
      <React application>
    infrastructure.py
      class UI:
        route53.HostedZone
        route53.CNAMERecord
        certificatemanager.Certificate
        cloudfront.Distribution
        s3.Bucket
        s3_assets.Asset
        cloudwatch.Metric
        cloudwatch.Alarm
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  service/
    runtime/ 
      <Express application + Dockerfile>
    infrastructure.py
      class Service:
        route53.HostedZone
        route53.CNAMERecord
        certificatemanager.Certificate
        waf.WebACL
        ec2.VPC
        ec2.Subnet
        ec2.RouteTable
        ec2.SecurityGroup
        elasticloadbalancingv2.NetworkLoadBalancer
        ecs.Cluster
        ecs.Service
        ecs.TaskDefinition
        ecr_assets.DockerImageAsset
        dynamodb.Table
        cloudwatch.Metric
        cloudwatch.Alarm
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  monitoring/
    infrastructure.py
      class Monitoring:
        route53.HostedZone
        route53.CNAMERecord
        cloudwatch.Dashboard
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  repository/
    infrastructure.py
      class Repository:
        route53.HostedZone
        route53.CNAMERecord
        certificatemanager.Certificate
        s3.Bucket
        cloudwatch.Metric
        cloudwatch.Alarm
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  infrastructure.py 
    class Components:
      UI
      Service
      Repository
      Monitoring
      servicecatalogappregistry.ResourceAssociation
metadata/
  infrastructure.py
    class Metadata:
      servicecatalogappregistry.Application
toolchain/
  infrastructure.py
    class Toolchain:
      pipelines.CodePipeline
      codebuild.Project
      servicecatalogappregistry.ResourceAssociation
app.py
  Metadata("Soteria-Metadata-Sandbox")
  Components("Soteria-Components-Sandbox")
  Toolchain("Soteria-Toolchain-Sandbox")
  
  Metadata("Soteria-Metadata-Management")
  Toolchain("Soteria-Toolchain-Management")
```

## Design evolution

### Architecture

As the functionality and the team grew, Soteria (documents storage) maintenance and evolution became harder and slower. The engineering team decided to split the Soteria application into smaller applications and keep Aegis (user management) and Athena (audit trail) as is. Soteria became 3 applications: 1/ Aphrodite (documents UI) 2/ Themis (documents service) 3/ Olympus (documents repository). The engineering team then moved existing components to the new applications and created dedicated components for each application (e.g. monitoring).

_Aphrodite_

* UI - Authenticates users and allows them to interact with the application
* Monitoring - Displays business and operational metrics

_Themis_

* Service - Generates protected links for upload and download
* Monitoring - Displays business and operational metrics

_Olympus_

* Repository - Stores the documents
* Monitoring - Displays business and operational metrics

Aphrodite and Themis interact with Aegis and Athena to sign in users and log user activity respectively.

### Architectural decision records

Engineering team captured the design in architectural decision records (ADRs).

_Aphrodite_

* UI infrastructure includes Route 53, CloudFront, S3, CloudWatch, X-Ray, and Service Catalog services. UI runtime includes React application.
* Monitoring infrastructure includes Route 53, CloudWatch, X-Ray, and Service Catalog services.

_Themis_

* Service infrastructure includes Route 53, Certificate Manager, WAF, EC2, Elastic Load Balancing, ECS, DynamoDB, CloudWatch, X-Ray, and Service Catalog services. Service runtime includes Express application.
* Monitoring infrastructure includes Route 53, CloudWatch, X-Ray, and Service Catalog services.

_Olympus_

* Repository infrastructure includes Route 53, S3, CloudWatch, X-Ray, and Service Catalog services.
* Monitoring infrastructure includes Route 53, CloudWatch, X-Ray, and Service Catalog services.

### Code structure

_Aphrodite_

Engineering team decided to create a single code repository for the Aphrodite application. The implementation includes components (UI and monitoring), toolchain (deployment pipeline and pull request build), and metadata (resources and attributes). Components, toolchain, and metadata deploy as a stack each. Components and toolchain resources are associated with the metadata.

Engineering team decided to create the following environments: 1/ per-builder sandbox environment 2/ management environment (toolchain and metadata) 3/ alpha, beta, gamma, and production environments (components). Toolchain deploys components to the alpha, beta, gamma and production environments.

```
Repo: aphrodite.git

components/
  ui/
    runtime/
      <React application>
    infrastructure.py
      class UI:
        route53.HostedZone
        route53.CNAMERecord
        certificatemanager.Certificate
        cloudfront.Distribution
        s3.Bucket
        s3_assets.Asset
        cloudwatch.Metric
        cloudwatch.Alarm
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  monitoring/
    infrastructure.py
      class Monitoring:
        route53.HostedZone
        route53.CNAMERecord
        cloudwatch.Dashboard
        xray.Group
        servicecatalogappregistry.AttributeGroup
        servicecatalogappregistry.AttributeGroupAssociation
  infrastructure.py 
    class Components:
      UI
      Monitoring
      servicecatalogappregistry.ResourceAssociation
metadata/
  infrastructure.py
    class Metadata:
      servicecatalogappregistry.Application
toolchain/
  infrastructure.py
    class Toolchain:
      pipelines.CodePipeline
      codebuild.Project
      servicecatalogappregistry.ResourceAssociation
app.py
  Metadata("Aphrodite-Metadata-Sandbox")
  Components("Aphrodite-Components-Sandbox")
  Toolchain("Aphrodite-Toolchain-Sandbox")
  
  Metadata("Aphrodite-Metadata-Management")
  Toolchain("Aphrodite-Toolchain-Management")
```

Themis and Olympus follow a similar approach to Aphrodite, so left out for brevity.
