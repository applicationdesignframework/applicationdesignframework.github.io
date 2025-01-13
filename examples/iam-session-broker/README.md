## Use case
A multi-tenant SaaS offering composes multiple applications. Applications use a shared IAM Session Broker library to scope user access to their tenant’s boundary. SaaS provider wants to build a shared IAM Session Broker application instead to reduce operational overhead and improve security posture. The application should initially support the ABAC authorization strategy.

### Business flow
* SaaS admin registers users for Yellow and Blue tenants
* Yellow user authenticates
* Yellow user downloads Yellow data

## Features and stories
Feature: Tenant isolation
* Job story: Scope the user access to their tenant’s boundary

## Requirements

### Business
* Require the applications to use ABAC with `${aws:PrincipalTag/`*`key`*`}` variable in policies for scoping access
* Require the applications to register and provide the following access metadata:
  * Access role name (e.g. `DocumentsAPIDataAccess`)
  * Session tag key (e.g. `TenantID`)
  * JWT claim name (e.g. `custom:tenant_id`)
  * [JSON Web Key (JWK) Set URL](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-step-2) (e.g. `https://cognito-idp.<Region>.amazonaws.com/<userPoolId>/.well-known/jwks.json`)
* Use the assumed-role session principal (service) role name for application name during registration
* Authorize the registration requests for assumed-role session principals in the same account
* Authorize the temporary security credentials requests for a registered application and a valid JWT
* Validate the JWT by public keys in the registered application JWK Set URL
* Return the scoped temporary security credentials by assuming the access role. Tag the session with the registered session tag key and JWT claim name value as the session tag key value.

### Technical
* Organize the access metadata by application name
* Generate SDKs for multiple programming languages from service specification
* Collect audit logs for security and compliance
* Use AWS STS to request temporary security credentials
* Cache the returned temporary security credentials to reduce latency
* Throttle applications on tenant-level to protect against noisy-neighbor

## Architecture

### Technical flow
![](https://user-images.githubusercontent.com/4362270/231122057-cfdbab53-ebc2-4344-8348-2b77c6e43e6b.jpg)

1. Yellow user authenticates using Identity Provider and gets a JWT
2. Yellow user accesses the Application with JWT to download Yellow data
3. Application calls IAM Session Broker to acquire Yellow-scoped temporary security credentials
4. IAM Session Broker verifies the JWT and returns Yellow-scoped temporary security credentials
5. Application returns Yellow data to the user

### Application boundaries
**Context**

We need to define application boundaires based on the technical flow.

**Decision**

Create [IAM Session Broker](https://github.com/applicationdesignframework/iam-session-broker) and [Identity Provider](https://github.com/applicationdesignframework/identity-provider) applications. IAM Session Broker returns tenant-scoped temporary security credentials based on JWT claims for registered applications. Identity Provider is not described in this example for brevity.

**Consequences**

IAM Session Broker is on the critical path for upstream applications. Hence, it should maintain the agreed upon service level objectives (SLOs). Users drive the requests volume to IAM Session Broker, because the user-agent provides the JWT. Hence, IAM Session Broker performance characteristics should take interactive flows as the baseline.

### IAM Session Broker components

**Context**

We need to define IAM Session Broker components based on the technical flow.

**Decision**

Create the following components:

![](https://github.com/user-attachments/assets/5105d66b-5939-4818-a3c2-139fa9f94a21)

API Gateway should authorize requests and throttle if needed to prevent the “noisy neighbor” problem. API Gateway should proxy all authorized and non-throttled requests to API. API should 1/ fetch access metadata from Access Database 2/ call Temporary Security Credentials Provider to assume the Service Role 3/ call Temporary Security Credentials Provider using the Service Role credentials to assume the access role 4/ return the scoped temporary security credentials.

_API Gateway_

Use Amazon API Gateway HTTP API with [IAM authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-access-control-iam.html). Use Lambda proxy integration for API. Leverage [usage plans](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) to prevent the “noisy neighbor” problem. Use the Lambda authorizer as the API key source ([example](https://catalog.us-east-1.prod.workshops.aws/workshops/026f84fd-f589-4a59-a4d1-81dc543fcd30/en-US/mod-5-usage/5d-use-authorizer)) and application name as the API key.

_API_

Use Lambda function for compute and [Powertools for AWS Lambda (Python)](https://docs.powertools.aws.dev/lambda/python/latest/) for application. Use Lambda [provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) to reduce latency. Cache applications scoped temporary security credentials to further reduce latency.

| Request | Description | Request body | Response |
| ------- | ----------- | ------------ | -------- |
| POST /applications | Register an application | {<br>&emsp;"AccessRoleName": "...",<br>&emsp;"SessionTagKey": "...",<br>&emsp;"JWTClaimName": "...",<br>&emsp;"JWKSetURL": "..."<br>} |  |
| GET /credentials?jwt=X | Return scoped temporary security credentials |  | {<br>&emsp;"AccessKeyId":"...",<br>&emsp;"SecretAccessKey":"...",<br>&emsp;"SessionToken":"..."<br>} |

_Access Database_

Use Amazon DynamoDB table. The table should store the registered access metadata.

Data model (designed using [NoSQL WorkBench for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)):

![Access Metadata data model](https://user-images.githubusercontent.com/4362270/231124010-d1d080e3-9a78-455f-a146-823d4bdd48d6.jpg)

_Temporary Security Credentials Provider_

Use [AWS Session Token Service (AWS STS)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html). Other options include AWS IAM Roles Anywhere and AWS IoT Core credential provider. Choose AWS STS because there is currently no requirement to support on-premises applications and/or certificate-based authentication use cases.

_Service Role_

Create `IAMSessionBroker` IAM role. Creating a dedicated Service Role enables flexibility for the IAM Session Broker architecture. Applications should trust the Service Role to assume their access role. The service role doesn’t have any permissions. Per requirements, applications and IAM Session Broker should be in the same account. The applications access role’s trust policy acts as an IAM resource-based policy. When a resource-based policy grants access to a principal in the same account, no additional identity-based policy is required ([documentation](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)).

*Note:* Applications access role trust policy uses the `IAMSessionBroker` role ID once saved. Deleting or altering the `IAMSessionBroker` role will require applications to update their access role’s trust policy to apply the new `IAMSessionBroker` role ID.

**Consequences**

To support cross-account scenarios, we would need to replace API Gateway HTTP API by API Gateway REST API, because HTTP API [doesn’t support](https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-iam-cross-account/) resource policies at the time of this writing.

API uses [role chaining](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-role-chaining): 1/ assume Service Role 2/ assume application access role. Role chaining limits the role session to a maximum of one hour. In the worst case scenario, the API will need to call Temporary Security Credentials Provider (AWS STS) every hour for a specific application.

### Service discovery

**Context**

We need to decide on a service discovery strategy to allow applications discover IAM Session Broker API endpoint without managing configuration files.

**Decision**

Use DNS for service discovery. Use the following naming convention for domain hierarchy:

```
<application>.<region>.<environment>.<product>.<top-level domain>
```

Example:

```
iam-session-broker.eu-west-1.gamma.saas-platform.example.com
```

Delegate the product sub-domain to a dedicated AWS account so that each product team can manage DNS zones for their applications. Using the above approach, applications can construct the IAM Session Broker API endpoint at deployment time.

**Consequences**

This approach supports cross-environment (account and Region) use cases by relying on naming convention.

## Code

### Toolchain
* Python: `3.9.11`
* AWS CDK CLI version: `2.175.1`
* AWS CDK library version: `2.175.1`

### Git repositories
* IAM Session Broker: https://github.com/applicationdesignframework/iam-session-broker
* Identity Provider: https://github.com/applicationdesignframework/identity-provider

### Project structure

**IAM Session Broker**

The project structure aligns with the architectural diagram, so that programmers can quickly find the code for each component.

```
service/
  api/
    app/
      main.py  # Business logic application - Powertools for AWS Lambda (Python)
    compute.py
  access_database.py
  api_gateway.py
  service_role.py
  service_stack.py
contstants.py
main.py  # Resources configuration application - AWS CDK
```

## Backlog


## Appendix

### API Gateway HTTP API with IAM authorizer Lambda proxy integration payload example
```
{
    "version": "2.0",
    "routeKey": "$default",
    "rawPath": "/applications",
    "rawQueryString": "",
    "headers": {
        "accept": "application/xml",
        "accept-encoding": "gzip, deflate",
        "authorization": "AWS4-HMAC-SHA256 Credential=ASIA3YC54HEJWX32ILMG/20230317/eu-west-1/execute-api/aws4_request, SignedHeaders=host;x-amz-date;x-amz-security-token, Signature=1b0063ba301f7668d5c7c982e505609db52ad83c3a879c311047acf6205cb760",
        "content-length": "0",
        "content-type": "application/json",
        "host": "d0cfuu2ujg.execute-api.eu-west-1.amazonaws.com",
        "user-agent": "python-requests/2.28.2",
        "x-amz-content-sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
        "x-amz-date": "20230317T170215Z",
        "x-amz-security-token": "<redacted>",
        "x-amzn-trace-id": "Root=1-64149d18-046d89152552d1da34770913",
        "x-forwarded-for": "85.250.125.159",
        "x-forwarded-port": "443",
        "x-forwarded-proto": "https"
    },
    "requestContext": {
        "accountId": "111111111111",
        "apiId": "d0cfuu2ujg",
        "authorizer": {
            "iam": {
                "accessKey": "ASIA3YC54HEJWX32ILMG",
                "accountId": "111111111111",
                "callerId": "AROA3YC54HEJ7ZID2KGLH:user@example.com",
                "cognitoIdentity": null,
                "principalOrgId": "aws:PrincipalOrgID",
                "userArn": "arn:aws:sts::111111111111:assumed-role/AWSReservedSSO_DeveloperAccess_073t3cf358b80610/user@example.com",
                "userId": "AROA3YC54HEJ7ZID2KGLH:user@example.com"
            }
        },
        "domainName": "d0cfuu2ujg.execute-api.eu-west-1.amazonaws.com",
        "domainPrefix": "d0cfuu2ujg",
        "http": {
            "method": "POST",
            "path": "/applications",
            "protocol": "HTTP/1.1",
            "sourceIp": "85.250.125.159",
            "userAgent": "python-requests/2.28.2"
        },
        "requestId": "B7170h_-DoEEPzA=",
        "routeKey": "$default",
        "stage": "$default",
        "time": "17/Mar/2023:17:02:16 +0000",
        "timeEpoch": 1679072536104
    },
    "isBase64Encoded": false
}
```

