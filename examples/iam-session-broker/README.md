# IAM Session Broker

## Use case
A multi-tenant SaaS offering composes multiple applications. Today, the applications use an IAM Session Broker library to scope user access to their tenant’s boundary. The SaaS provider wants to build an IAM Session Broker service instead to reduce operational overhead and improve security posture. The service should initially support ABAC authorization strategy.

Definitions:
* Application service principal: A principal application uses to interact with the IAM Session Broker service.
* Application access principal: A principal application uses to interact with its data.
* IAM Session Broker service principal: A principal IAM Session Broker service uses to provide scoped credentials for application access principal.

### Diagram
* Green - Registration business flow.
* Orange - Data access business flow.

![](/images/iam-session-broker-use-case.svg)

### Business flows

#### Registration
**Steps**
1. Application registers with the IAM Session Broker service.
2. SaaS admin registers user for the Yellow tenant.

**Requirements**
* Applications should register and provide the following access metadata:
  * Access principal role name (e.g. `AppAccess`)
  * Session tag key (e.g. `TenantID`)
  * JWT claim name (e.g. `custom:tenant_id`)
  * [JSON Web Key (JWK) Set URL](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-step-2) (e.g. `https://cognito-idp.<Region>.amazonaws.com/<userPoolId>/.well-known/jwks.json`)
* Applications should use application service principal role name as application name for registration.
* Applications should use [ABAC](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html) with `${aws:PrincipalTag/`*`key`*`}` variable in access principal policies to enforce tenant boundary.
* Automatically authorize registration for application service principals in the IAM Session Broker environment.

#### Data access
**Steps**
1. Yellow user authenticates.
2. Yellow user accesses the Application to download Yellow data.
3. Application sends Yellow tenant credentials to IAM Session Broker to get Yellow-scoped temporary security credentials.
4. Application uses the Yellow-scoped temporary security credentials to return the data.

**Requirements**
* Authorize temporary security credentials requests for a registered application and a valid JWT.
* Validate the provided JWT using public keys of the registered application JWK Set URL.
* Return the scoped temporary security credentials by assuming the registered application access principal role. Tag the role session with the registered session tag key and JWT claim name value.

## Architecture

### Diagram
* Data access technical flow. Registration technical flow is out of scope for brevity.

![](/images/iam-session-broker-system-architecture.svg)

### Technical flows
#### Registration
**Steps**
* Application registers with IAM Session Broker service by authenticating and providing the access principal role.
* SaaS admin authenticates and registers users for Yellow and Blue tenants.

**Requirements**
* Organize the access metadata by application name.

#### Data access
**Steps**
1. Yellow user authenticates using Identity Provider and client receives a JWT.
2. Yellow user accesses the Application with the JWT to download Yellow data.
3. Application sends Yellow tenant JWT to IAM Session Broker.
4. IAM Session Broker verifies the JWT, assumes the application access principal role, and returns Yellow-scoped temporary security credentials.
* Application uses the Yellow-scoped temporary security credentials to return the data.

**Requirements**
1. Generate SDKs for multiple programming languages from service specification.
2. Collect audit logs for security and compliance.
3. Cache the returned temporary security credentials to reduce latency.
4. Throttle applications on tenant-level to protect against noisy-neighbor.

### Architectural decision records (ADRs)
#### Application boundaries
**Context**

We need to define application boundaries.

**Decision**

Create [IAM Session Broker](https://github.com/applicationdesignframework/iam-session-broker) and [Identity Provider](https://github.com/applicationdesignframework/identity-provider) applications. IAM Session Broker returns tenant-scoped temporary security credentials based on JWT claims for registered applications. Identity Provider is not described in this example for brevity.

**Consequences**

IAM Session Broker is on the critical path for upstream applications. Hence, it should maintain the agreed upon service level objectives (SLOs). Users drive the requests volume to IAM Session Broker, because the user-agent provides the JWT. Hence, IAM Session Broker performance characteristics should take interactive flows as the baseline.

#### IAM Session Broker components and technologies

**Context**

We need to define IAM Session Broker components and choose technologies.

**Decision**

![](/images/iam-session-broker-application-architecture-technology.svg)

API Gateway should authorize requests and throttle if needed to prevent the “noisy neighbor” problem. API Gateway should proxy all authorized and non-throttled requests to API. The API should: 1/ fetch access metadata from access database 2/ call temporary security credentials provider to get the service principal role credentials 3/ call temporary security credentials provider using the service principal role credentials to get the application access principal role credentials 4/ return the scoped application access principal role credentials.

_API Gateway_

Use Amazon API Gateway HTTP API with [IAM authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-access-control-iam.html). Use Lambda proxy integration for API. Leverage [usage plans](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) to prevent the “noisy neighbor” problem. Use the Lambda authorizer as the API key source ([example](https://catalog.us-east-1.prod.workshops.aws/workshops/026f84fd-f589-4a59-a4d1-81dc543fcd30/en-US/mod-5-usage/5d-use-authorizer)) and application name as the API key.

_API_

Use Lambda function for compute and [Powertools for AWS Lambda (Python)](https://docs.powertools.aws.dev/lambda/python/latest/) for application. Use Lambda [provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) to reduce latency. Cache scoped temporary security credentials to reduce latency.

| Request | Description | Request body | Response |
| ------- | ----------- | ------------ | -------- |
| POST /applications | Register an application | {<br>&emsp;"AccessPrincipalRoleName": "...",<br>&emsp;"SessionTagKey": "...",<br>&emsp;"JWTClaimName": "...",<br>&emsp;"JWKSetURL": "..."<br>} |  |
| GET /credentials?jwt=X | Return scoped temporary security credentials |  | {<br>&emsp;"AccessKeyId":"...",<br>&emsp;"SecretAccessKey":"...",<br>&emsp;"SessionToken":"..."<br>} |

_Access Database_

Use Amazon DynamoDB table. The table should store the registered access metadata.

Data model (designed using [NoSQL Workbench](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)):

![](/images/iam-session-broker-access-database-model.png)
([source](/models/iam-session-broker-access-database.json))

_Temporary Security Credentials Provider_

Use [AWS Session Token Service (AWS STS)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html). Other options include AWS IAM Roles Anywhere and AWS IoT Core credential provider. Choose AWS STS because there is currently no requirement to support on-premises applications and/or certificate-based authentication use cases.

_Service Principal_

Create `IAMSessionBroker` IAM role representing the service principal. Creating a dedicated service principal role enables flexibility for the IAM Session Broker architecture. Applications should trust the service principal role to assume their access principal role. The service principal role doesn’t have any permissions. Per requirements, applications and IAM Session Broker should be in the same account. The application access principal role trust policy is an IAM resource-based policy. When a resource-based policy grants access to a principal in the same account, no additional identity-based policy is required for the principal role ([documentation](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)).

*Note:* Application access principal role trust policy uses the `IAMSessionBroker` role ID once saved. Deleting or altering the `IAMSessionBroker` role will require applications to update their access principal role trust policy to apply the new `IAMSessionBroker` role ID.

**Consequences**

To support cross-account scenarios, we would need to replace API Gateway HTTP API by API Gateway REST API, because HTTP API [doesn’t support](https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-iam-cross-account/) resource policies at the time of this writing.

API uses [role chaining](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-role-chaining): 1/ get service principal role credentials 2/ get application access principal role credentials. Role chaining limits the role session to a maximum of one hour. In the worst case scenario, the API will need to call temporary security credentials provider (AWS STS) every hour for the related application.

#### Service discovery

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

**Consequences**

This approach supports cross-environment (account and Region) use cases by relying on naming convention.

## Code

### Toolchain
* Python: `3.9.11`
* AWS CDK CLI version: `2.175.1`
* AWS CDK library version: `2.175.1`

### Git repositories
* IAM Session Broker: [iam-session-broker](https://github.com/applicationdesignframework/iam-session-broker)
* Identity Provider: [identity-provider](https://github.com/applicationdesignframework/identity-provider)

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
  service_principal.py
  service_stack.py
constants.py
main.py  # Resources configuration application - AWS CDK
```

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

