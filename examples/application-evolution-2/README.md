# Application evolution 2

This example describes changes in deployment and ownership as part of company growth. It doesn't go into details of stories, requirements, architecture, architectural decision records and code structure.

## Initial design

AnyCompany product team wants to build a DocuStar product that allows enterprises to store, view, edit, and share documents. Engineering team decided to create a single Soteria application with 4 components: 1/ documents UI 2/ documents service 3/ user management service 4/ IAM session broker service. Soteria application also includes toolchain (deployment pipeline and pull request build) and metadata (resources and attributes). Components, toolchain, and metadata deploy as a stack each. Components and toolchain resources are associated with the metadata.

### Architecture

![design-1-architecture](https://github.com/alexpulver/adf/assets/4362270/b32e2b8e-fa0f-4544-b9ea-45cbbc948d34)

### Deployment

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/fafe4a87-0853-4d6d-b4cb-54971dc367e8" width="35%">
</p>

## Deployment evolution

There were several occasions where code changes inadvertently deleted data resources (e.g. documents repository). The team had snapshots, but it took time to restore. The engineering team decided to split the components into discrete stacks and add termination protection for the sensitive resources to decrease blast radius.

### Architecture

![design-2-architecture](https://github.com/alexpulver/adf/assets/4362270/d9ff0a12-927f-44a8-8702-47d65dfd6077)

### Deployment

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/a213c7c0-cc63-462f-978a-341dbd43e331" width="35%">
</p>

## Ownership evolution
As the company grew, it established more teams and also decided to create an internal platform (DocuGuard) for the shared user management and IAM session broker services. There was now a dedicated team for each of the components (documents UI, documents service, user management service, IAM session broker service). The company decided to create a dedicated application for each of these components and assign ownership to the respective teams.

### Architecture

![design-3-architecture](https://github.com/alexpulver/adf/assets/4362270/5460adff-62b9-4a01-af4d-7ae03f3b51e0)

### Deployment

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/b93a1bc6-311c-4229-a68a-b89530e86d7d" width="65%">
</p>
