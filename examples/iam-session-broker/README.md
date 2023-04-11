## Use case

A multi-tenant SaaS composes multiple applications. Applications use a shared IAM Session Broker library to scope user access to their tenant’s boundary. SaaS provider wants to build a shared IAM Session Broker service instead to reduce operational overhead and improve security posture. The service should initially support the ABAC authorization strategy.

## Stories

* Scope the user access to their tenant’s boundary

## Requirements

### Flows

Scope the user access to their tenant’s boundary

* SaaS admin registers users for Yellow and Blue tenants
* Yellow user authenticates
* Yellow user downloads Yellow data
* Yellow user gets access denied when trying to download Blue data

### Functional

* Require the applications to use ABAC with `${aws:PrincipalTag/`*`key`*`}` variable in policies for scoping access
* Require the applications to register and provide the following access metadata:
  * Access role name (e.g. `DocumentsAPIDataAccess`)
  * Session tag key (e.g. `TenantID`)
  * JWT claim name (e.g. `custom:tenant_id`)
  * [JSON Web Key (JWK) Set URL](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-step-2) (e.g. `https://cognito-idp.<Region>.amazonaws.com/<userPoolId>/.well-known/jwks.json`)
* Organize the access metadata by application name
* Use the assumed-role session principal (service) role name for application name during registration
* Authorize the registration requests for assumed-role session principals in the same account
* Authorize the temporary security credentials requests for a registered application and a valid JWT
* Validate the JWT by public keys in the registered application JWK Set URL
* Return the scoped temporary security credentials by assuming the access role. Tag the session with the registered session tag key and JWT claim name value as the session tag key value.

### Non-functional

* Generate SDKs for multiple programming languages from service specification
* Collect audit logs for security and compliance
* Use AWS STS to request temporary security credentials
* Cache the returned temporary security credentials to reduce latency
* Throttle applications on tenant-level to protect against noisy-neighbor

## Architecture

### Flows and components

**Context**

We need to define components and describe how they communicate and interact based on the [Flows](#flows).

**Decision**

Scope the user access to their tenant’s boundary

![Scope the user access to their tenant’s boundary](https://user-images.githubusercontent.com/4362270/231122057-cfdbab53-ebc2-4344-8348-2b77c6e43e6b.jpg)

1. Yellow user authenticates using the Identity Provider and gets a JWT
2. Yellow user accesses the Application with JWT to download Yellow data
3. Application calls IAM Session Broker to acquire Yellow-scoped temporary security credentials
4. IAM Session Broker verifies the JWT and returns Yellow-scoped temporary security credentials
5. Application returns Yellow data to the user

Create [IAM Session Broker API](#iam-session-broker-api) component that returns tenant-scoped temporary security credentials for applications. IAM Session Broker API should have a dedicated Git repository and a pipeline to reduce blast radius and increase delivery performance. [Identity Provider API](#identity-provider-api) is out of scope but included for convenience in the Appendix.

**Consequences**

IAM Session Broker API is on the critical path for upstream applications. Hence, it should maintain the agreed upon service level objectives (SLOs). Users drive the requests volume to IAM Session Broker API, because the user-agent provides the JWT. Hence, IAM Session Broker API performance characteristics should take interactive flows as the baseline.

### Service discovery

**Context**

We need to decide on a service discovery strategy to allow applications discover IAM Session Broker API endpoint without managing configuration files.

**Decision**

Use DNS for service discovery. Use the following naming convention for domain hierarchy:

```
<component>.<region>.<account>.<application>.<top-level domain>
```

Example:

```
api.eu-west-1.111111111111.iam-session-broker.example.com
```

Delegate the application sub-domain to a dedicated AWS account so that each application can manage their DNS zones. Using the above approach, applications can construct the IAM Session Broker API endpoint at deployment time using their account and Region values.

**Consequences**

This approach supports the use case where all applications deploy to the same environment (account and Region). Cross-environment application deployments would require a configuration file, because an application wouldn’t be able to infer the environment of the dependencies.

### IAM Session Broker API

**Context**

We need to build an IAM Session Broker API that returns scoped temporary security credentials based on JWT claims.

**Decision**

![IAM Session Broker API](https://user-images.githubusercontent.com/4362270/231123426-c3d8567e-7071-4dd5-8322-02cd4bb381b5.jpg)

Gateway should authorize requests and throttle if needed to prevent the “noisy neighbor” problem. Gateway should proxy all authorized and non-throttled requests to Credentials Manager. Credentials Manager should 1/ fetch Access Metadata 2/ call Temporary Security Credentials Provider to assume the Service Role 3/ call Temporary Security Credentials Provider using the Service Role credentials to assume the access role 4/ return the scoped temporary security credentials.

_Gateway_

Use Amazon API Gateway HTTP API with [IAM authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-access-control-iam.html). Use proxy integration for Credentials Manager. Leverage [usage plans](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) to prevent the “noisy neighbor” problem. Use the Lambda authorizer as the API key source ([example](https://catalog.us-east-1.prod.workshops.aws/workshops/026f84fd-f589-4a59-a4d1-81dc543fcd30/en-US/mod-5-usage/5d-use-authorizer)) and application name as the API key.

_Credentials Manager_

Use Lambda function and [Lambda Powertools for Python](https://awslabs.github.io/aws-lambda-powertools-python/). Use Lambda [provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) to reduce latency. Cache applications scoped temporary security credentials to further reduce latency.

| Request | Description | Request body | Response |
| ------- | ----------- | ------------ | -------- |
| POST /applications | Register an application | {<br>&emsp;"AccessRoleName": "...",<br>&emsp;"SessionTagKey": "...",<br>&emsp;"JWTClaimName": "...",<br>&emsp;"JWKSetURL": "..."<br>} |  |
| GET /credentials?jwt=X | Return scoped temporary security credentials |  | {<br>&emsp;"AccessKeyId":"...",<br>&emsp;"SecretAccessKey":"...",<br>&emsp;"SessionToken":"..."<br>} |

_Access Metadata_

Use Amazon DynamoDB. Create an `AccessMetadata` DynamoDB table. The table should store the registered access metadata.

Data model (designed using [NoSQL WorkBench for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)):

![Access Metadata data model](https://user-images.githubusercontent.com/4362270/231124010-d1d080e3-9a78-455f-a146-823d4bdd48d6.jpg)

_Temporary Security Credentials Provider_

Use [AWS Session Token Service (AWS STS)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html). Other options include AWS IAM Roles Anywhere and AWS IoT Core credential provider. Choose AWS STS because there is currently no requirement to support on-premises applications and/or certificate-based authentication use cases.

_Service Role_

Create `IAMSessionBrokerAPI` IAM role. Creating a dedicated Service Role enables flexibility for the IAM Session Broker API architecture. Applications should trust the Service Role to assume their access role. The service role doesn’t have any permissions. Per requirements, applications and IAM Session Broker API should be in the same account. The applications access role’s trust policy acts as an IAM resource-based policy. When a resource-based policy grants access to a principal in the same account, no additional identity-based policy is required ([documentation](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)).

*Note:* Applications access role trust policy uses the `IAMSessionBrokerAPI` role ID once saved. Deleting or altering the `IAMSessionBrokerAPI` role will require applications to update their access role’s trust policy to apply the new `IAMSessionBrokerAPI` role ID.

**Consequences**

To support cross-account scenarios, would need to replace API Gateway HTTP API by API Gateway REST API, because HTTP API [doesn’t support](https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-iam-cross-account/) resource policies at this time.

IAM Session Broker API uses [role chaining](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-role-chaining): 1/ assume Service Role 2/ assume application access role. Role chaining limits the role session to a maximum of one hour. In the worst case scenario, the IAM Session Broker API will need to call Temporary Security Credentials Provides (AWS STS) every hour for a specific application.

## Implementation

### **Toolchain**

Python: `3.9.11`

AWS CDK Toolkit (CLI) and AWS CDK Construct Library: `2.69.0`

Project template: https://github.com/aws-samples/aws-cdk-project-structure-python

### **Git repositories**

IAM Session Broker API: `iam-session-broker-api`

Identity Provider API: `identity-provider-api`

### **Project structure**

_IAM Session Broker API_
```
iam_session_broker_api/
  credentials_manager/
    runtime/
      <custom code>
  component.py 
    IAMSessionBrokerAPI (Stack construct) 
      Gateway (API Gateway construct)
      CredentialsManager (Lambda construct)
      AccessMetadata (DynamoDB construct)
      ServiceRole (IAM construct)
```

_Identity Provider API_
```
identity_provider_api/ 
  component.py 
    IdentityProviderAPI (Stack construct) 
      IdentityStore (Cognito construct)
```

## Backlog


## Appendix

### Identity Provider API

**Context**

We need to design customer identity and access management solution. The solution should: 1/ allow each builder to test the identity provider in their sandbox account 2/ support multiple pre-production and production environments 3/ enable a sign in UI endpoint discovery without managing configuration files.

**Decision**

![Identity Provider API](https://user-images.githubusercontent.com/4362270/231124377-3bb40e55-b1f5-4289-94b2-9db89428b265.jpg)

_Identity Store_

Use Amazon Cognito as the identity store with a single user pool for all tenants and Cognito hosted UI for sign in. Add `tenant_id` and `role` custom attributes to provide access control context for applications. SaaS admin should register the users manually (there is not a self-signup process yet). After authenticating the user, Amazon Cognito should return a JWT with SaaS identity (include user ID and tenant ID).

Another option would be to use a Cognito user pool per tenant. Then we would have to build a custom main sign in UI that would route the users to their tenant’s user pool hosted UI. We cannot use Cognito hosted UI for the main sign in screen and federate to tenants’ Cognito user pools, because Cognito doesn't support IdP-initiated sign in. To focus on the access control use case, we chose the first option of a single Cognito user pool.

Use the following domain prefix naming convention for [Cognito domain](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-assign-domain-prefix.html):

```
<account>-<top-level sub-domain>
```

Use top-level sub-domain because all applications should authenticate users with a single identity provider.

Cognito domain example:

* Account: `111111111111`
* Top-level sub-domain: `example`
* Region: `eu-west-1`

```
111111111111-example.auth.eu-west-1.amazoncognito.com
```

Applications should construct sign in UI endpoint for the same environment using their account, Region, and top-level sub-domain values.

Register users using the AWS CLI and set `tenant_id` and `role` attributes for each user. The command is:

```
aws cognito-idp admin-create-user \
  --user-pool-id USER_POOL_ID\
  --username EMAIL\
  --user-attributes \
    Name=email,Value=EMAIL \
    Name=email_verified,Value=True \
    Name=custom:tenant_id,Value=TENANT_ID \
    Name=custom:role,Value=ROLE
```

**Consequences**

Clients need to perform client-side filtering of users by tenant ID, because Cognito doesn’t support searching by custom attributes (which `tenant_id` is). It’s not possible to implement different security configurations for each tenant. See [Multi-tenant application best practices](https://docs.aws.amazon.com/cognito/latest/developerguide/multi-tenant-application-best-practices.html) in Cognito documentation for more details.

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

