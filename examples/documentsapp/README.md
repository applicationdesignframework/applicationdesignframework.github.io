## Use case

DocumentsApp B2B SaaS application that allows members to manage (add, list, share) documents. We assume that market research showed the need in this product.

## Stories

_As DocumentsApp Owner, I want to register users_

_As a Member, I want to add a document_

_As a Member, I want to see list of my documents and documents shared with me in the documents UI_

_As ExampleCorp Admin, I want to see the list of all ExampleCorp documents_

_As ExampleCorp Member, I want to share a document with another ExampleCorp Member_

## Requirements

### Flows

_As DocumentsApp Owner, I want to register users_

* DocumentsApp Owner uses a command line interface (CLI) to register users

_As a Member, I want to add a document_

* Member navigates to the documents UI and authenticates
* Member clicks on `Add` button
* Member enters document name and clicks `Submit` button

_As a Member, I want to see list of my documents and documents shared with me in the documents UI_

* Member navigates to the documents UI and authenticates
* Member sees the list of documents they own in “My Documents” table view
* Member sees the list of documents shared with them “Shared With Me” table view

![](https://user-images.githubusercontent.com/4362270/227050513-9fe91f8b-52b3-48b3-9b2c-c94d6eaf8af4.png)

_As ExampleCorp Admin, I want to see the list of all ExampleCorp documents_

* Admin navigates to the documents UI and authenticates
* Admin sees the documents of all ExampleCorp users in a table view of the documents UI

_As ExampleCorp Member, I want to share a document with another ExampleCorp Member_

* Member A selects the document they want to share
* Member A clicks on the `Share` button
* Member A sees the list of all ExampleCorp Members 
* Member A selects Member B to share the document with and clicks on the `Submit` button
* Member B can see the document Member A shared with them
* Member A clicks `Unshare` button
* Member B can no longer see the document

### User

* DocumentsApp Owner should register all users manually with tenant ID and role attributes
* Members cannot share documents with Admins
* Admins cannot add documents
* Admins should see the documents of all tenant Members along with the Member email

### Functional

* DocumentsApp entry point should be www.documentsapp.com and show the documents UI
* DocumentsApp should authenticate users with username and password
* DocumentsApp should allowlist the following tenant IDs: `ExampleCorp` and `AnyCompany`
* DocumentsApp should support the following roles: `Member` and `Admin`
* ExampleCorp users should not be able to see AnyCompany documents
* Documents UI should redirect unauthenticated users to the sign in page
* Documents UI should show the email of the signed in user
* Documents UI should have a sign out button
* Documents UI should show a list of documents owned by the signed in user under `My Documents` view
* Documents UI should show `Share` or `Unshare` button next to document name in `My Documents` view
* Documents UI `Shared With Me` view should show document owner email
* Documents UI should allow sharing a document with one user at any given moment

### Non-functional

* DocumentsApp should manage all users internally
* Signed in user should bear a SaaS identity. The identity should include tenant ID and role.

## Architecture

### Application

#### Context

We need to create application architecture to address the requirements. The architecture should define component boundaries and integration patterns between them.

#### Decision

Introduce 4 components: 1/ Documents UI 2/ Documents API 3/ Identity Provider API 4/ Token Vending Machine API. Decouple the Identity Provider API from other components because 1/ both Documents UI and Documents API need to interact with it 2/ it has different risk profile and change cadence. Decouple Documents UI from Documents API because they have difference change cadence. Decouple Token Vending Machine API from Documents API because it has different risk profile and change cadence. Create a Git repository and pipeline for each component to reduce blast radius and increase delivery performance.

![](https://user-images.githubusercontent.com/4362270/227508807-156123e1-6278-4448-a50b-b275a498f41c.png)

#### Consequences

Will need to create application-level [operational dashboard](https://aws.amazon.com/builders-library/building-dashboards-for-operational-visibility/) to capture cross-component aggregate metrics. Components interact over network which requires additional logic (e.g. retries, circuit breakers) and tooling (e.g. tracing). 

### Flows

#### Context

We need to define flows between the application components.

#### Decision

_As DocumentsApp Owner, I want to register users_

The owner should register users with AWS CLI. The admin should set the following custom attributes for each user: `tenant_id` (`ExampleCorp` or `AnyCompany`) and `role` (`Member` or `Admin`). The command is:

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

_As a Member, I want to add a document_

When the Member clicks on `Add Document` button, the Documents UI will prompt for document name, and use it as document ID. When the Member clicks `Submit` button, the Documents UI should call the Documents API to add the document.

_As a Member, I want to see list of my documents and documents shared with me in the documents UI_

When the Member browses to the Documents UI, the UI should call the Documents API to get the list of documents owned by and shared with the Member. The Documents API should also return documents metadata that contains owners and sharing permissions. The list of documents should then be displayed.

_As ExampleCorp Admin, I want to see the list of all ExampleCorp documents_

When the Admin browses to the Documents UI, the UI should call the Documents API to get the list of all ExampleCorp documents. The Documents API should also return documents metadata that contains owners. The list of documents should then be displayed.

_As ExampleCorp Member, I want to share a document with another ExampleCorp Member_

When the Member wants to share a document, the Documents UI should call the Documents API to get the list of ExampleCorp users. When the Member selects the user to share the document with and submits, the Documents UI should call the Documents API with document and sharee details.

#### Consequences

We should implement a DNS strategy and a service discovery mechanism so that components can perform redirects and make API calls.

### DNS strategy

#### Context

We need to decide on DNS strategy that would: 1/ allow each builder to test the full application in their sandbox account 2/ support multiple pre-production and production environments 3/ enable cross-component service discovery without managing configuration files and making network calls to a service registry.

#### Decision

Use the following naming convention for domain hierarchy:

```
<component>.<region>.<account>.<application>.<top-level domain>
```

Examples:

```
documents-ui.eu-west-1.111111111111.documentsapp.com
documents-api.eu-west-1.111111111111.documentsapp.com
```

Delegate the environment (`111111111111`) subdomain to the environment’s AWS account. Each component knows the application name at design time. The component can construct the endpoint for dependencies at deployment time using its own account and Region values.

#### Consequences

This approach supports applications where all components deploy to the same environment (account and Region). Cross-environment component deployments would require a configuration file or service registry, because a component wouldn’t be able to infer the environment of the dependencies. For production scenario, need to create a CNAME record for Documents UI domain, e.g. `www.documentsapp.com`.

### Documents UI

#### Context

TBD

#### Decision

TBD

#### Consequences

TBD

### Documents API

#### Context

TBD

#### Decision

TBD

#### Consequences

TBD

### Identity Provider API

#### Context

We need to design customer identity and access management solution. The solution should: 1/ allow each builder to test the component in their sandbox account 2/ support multiple pre-production and production environments 3/ enable sign in UI endpoint discovery without managing configuration files and making network calls to a service registry for clients in the same environment (account and Region).

#### Decision

Use Amazon Cognito as the identity provider with a single user pool for all tenants and Cognito hosted UI for sign in. Store the user `tenant_id` (`ExampleCorp` or `AnyCompany`) and `role` ( `Member` or `Admin`) as custom attributes, because we need to perform access control in the application based on this information. DocumentsApp Owner should register users (there is not self-signup process yet). They should assign a role to the user. After authenticating the user, Amazon Cognito should return a JWT with SaaS identity - user ID, tenant ID and role.

Another option would be to use a Cognito user pool per tenant. Then we would have to build a custom main sign in UI that would route the users to their tenant’s user pool hosted UI. We cannot use Cognito hosted UI for the main sign in screen and federate to tenants’ Cognito user pools, because Cognito doesn't support IdP-initiated sign in. To focus on the access control use case we chose the first option of a single Cognito user pool.

Use the following domain naming convention for [Cognito domain](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-assign-domain-prefix.html):

```
<account>-<application>.auth.<region>.amazoncognito.com
```

Example:

```
111111111111-documentsapp.auth.eu-west-1.amazoncognito.com
```

Clients should construct sign in UI endpoint for the same environment using their account, Region, and application subdomain values. This will allow to easily deploy Cognito user pool to multiple environments.

#### Consequences

Clients need to perform client-side filtering of users by tenant ID, because Cognito doesn’t support searching by custom attributes (which `tenant_id` is). Each client needs to implement cross-tenant access control.

It’s not possible to implement different configurations for each tenant. See [Multi-tenant application best practices](https://docs.aws.amazon.com/cognito/latest/developerguide/multi-tenant-application-best-practices.html) in Cognito documentation for more details.

### Token Vending Machine API

#### Context

We need to build a Token Vending Machine API (TVM API) that implements IAM session broker to vend temporary credentials. Services’ runtime code should not have any access to data at deployment time. The runtime code should call TVM API to get temporary scoped credentials for access to data. This approach should improve security and make audits easier. We still want services to define their own roles and policies to avoid bottlenecks in development. The policies should use `${aws:PrincipalTag/KEY}` to scope access. Services should be able to control access scope based on the ID token claims of the authenticated user.

#### Decision

The TVM API should vend temporary credentials for a service access role. It should tag the role session based on predefined ID token claim value of the authenticated user to implement access scoping. Services should allow the TVM API Service Principal role to assume the service access role.

Services should register with the TVM API before asking for temporary credentials. For initial implementation, the TVM API should trust any role in the same account to register a service. The TVM API should authenticate the caller using AWS Identity and Access Management (AWS IAM), authorize the request if the caller from the same account, and use the caller role name (e.g. `DocumentsAPI`) as the service principal role name for registration. 

Services should provide the following information during registration:

* Service access role name (e.g. `DocumentsAPIDataAccess`). TVM API should vend temporary credentials for this role. Current implementation should support one service access role per service principal role for simplicity. The service should allow TVM API Service Principal to assume the service access role.
* [JSON Web Key Set URL](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-step-2) (e.g. `https://cognito-idp.REGION.amazonaws.com/USER_POOL_ID/.well-known/jwks.json`). TVM API should use JWKS to verify the ID token.
* Session tag key (e.g. `TenantID`). TVM API will tag the session using this key. Current implementation should support a single tag key per service principal role for simplicity. 
* ID token claim name (e.g. `custom:tenant_id`). TVM API will use the claim value as session tag value.

TVM API should authorize requests to vend credentials if the 1/ service principal role is registered in TVM API 2/ service provides a valid ID token. TVM API requires a valid ID token instead of relying on arbitrary scope parameter for defense-in-depth. The threat model is to prevent an attacker who gained access to service runtime code from gaining cross-tenant access by means of manipulating the scope parameter value. TVM API should assume the service access role registered by the service and tag the session with the registered session tag key. TVM API should use the registered ID token claim as session tag value.

_Service Principal_

Use IAM role. The TVM API should create `TokenVendingMachineAPI` role. TVM should use this role to assume the services’ access role.

*Note:* Services’ trust policies use the `TokenVendingMachineAPI` role ID once saved. Deleting or altering the `TokenVendingMachineAPI` role will require services to update their trust policies to apply the new `TokenVendingMachineAPI` role ID.

_Gateway_

Use Amazon API Gateway HTTP API with [IAM authorization](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-access-control-iam.html) and proxy all requests to IAM Session Broker. 

_IAM Session Broker_

Use Lambda function and [AWS Lambda Powertools for Python](https://awslabs.github.io/aws-lambda-powertools-python/).

| Request | Description | Request body | Response |
| ------- | ----------- | ------------ | -------- |
| POST /services | Register service | {<br>&emsp;"ServiceAccessRoleName": "...",<br>&emsp;"JSONWebKeySetURL": "...",<br>&emsp;"SessionTagKey": "...",<br>&emsp;"IDTokenClaimName": "..."<br>} |  |
| GET /credentials?id-token=X | Vend temporary credentials |  | {<br>&emsp;"SecretAccessKey":"...",<br>&emsp;"SessionToken":"...",<br>&emsp;"Expiration":"...",<br>&emsp;"AccessKeyId":"..."<br>} |

_Access Metadata_

Use Amazon DynamoDB. Create a `AccessMetadata` DynamoDB table. The table should store the services access metadata.

Data model (created using [NoSQL WorkBench for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)):

![](https://user-images.githubusercontent.com/4362270/227051229-fba18c1b-7fa9-401d-a9a4-bcdd657909e1.jpg)

#### Consequences

To support vending credentials for cross-account scenarios, would need to replace API Gateway HTTP API by API Gateway REST API, because HTTP API [doesn’t support](https://aws.amazon.com/premiumsupport/knowledge-center/api-gateway-iam-cross-account/) resource policies at this time.

Vending credentials should be a low-latency operation. Because we use Lambda for compute, consider [provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) to reduce latency.

## Implementation

### Toolchain

Python: `3.9.11`

AWS CDK Toolkit (CLI) and AWS CDK Construct Library: `2.69.0`

### Project scaffolding

Template: https://github.com/aws-samples/aws-cdk-project-structure-python

Application name: `DocumentsApp`

### Git repositories

Documents UI: `documentsapp-documents-ui`

Documents API: `documentsapp-documents-api`

Identity Provider API: `documentsapp-identity-provider-api`

Token Vending Machine API: `documentsapp-token-vending-machine-api`

### AWS CDK project structure

_Documents UI_

TBD


_Documents API_

TBD

_Identity Provider API_
```
identity_provider_api/ 
  component.py 
    IdentityProviderAPI (Stack construct) 
      IdentityStore (Cognito construct)
```

_Token Vending Machine API_
```
token_vending_machine_api/
  iam_session_broker/
    runtime/
      <custom code files>
  component.py 
    TokenVendingMachineAPI (Stack construct) 
      Gateway (API Gateway construct), IAMSessionBroker (Lambda construct), AccessMetadata (DynamoDB construct)
```

## Backlog
TBD
