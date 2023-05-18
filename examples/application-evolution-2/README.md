# Application evolution 2

This example describes changes in deployment and ownership as part of company growth. It doesn't go into details of stories, requirements, architecture, architectural decision records and code structure.

## Initial design

AnyCompany product team wants to build a DocuStar product that allows enterprises to store, view, edit, and share documents. Engineering team decided to create a single Soteria application with 4 components: 1/ documents UI 2/ documents service 3/ user management service 4/ IAM session broker service. Soteria application also includes toolchain (deployment pipeline and pull request build) and metadata (resources and attributes). Components, toolchain, and metadata deploy as a stack each. Components and toolchain resources are associated with the metadata.

### Architecture

![design-1-architecture](https://github.com/alexpulver/adf/assets/4362270/8f699306-92d4-40a7-8b23-5b20895d99d6)

### Deployment

<img src="https://github.com/alexpulver/adf/assets/4362270/818f5ef1-f4f4-47b3-8e94-ed2fa25670cc" width="35%">

## Deployment evolution

There were several occasions where code changes inadvertently deleted data resources (e.g. documents repository). The team had snapshots, but it took time to restore. The engineering team decided to split the components into discrete stacks and add termination protection for the sensitive resources to decrease blast radius.

### Architecture

![design-2-architecture](https://github.com/alexpulver/adf/assets/4362270/794f40b2-c948-479f-9b37-08cc5b9dbfa7)

### Deployment

<img src="https://github.com/alexpulver/adf/assets/4362270/e2b56bda-4eff-44ed-bc0b-38046d968d11" width="35%">

## Ownership evolution
As the company grew, it established more teams and also decided to create an internal platform (DocuGuard) for the shared user management and IAM session broker services. There was now a dedicated team for each of the components (documents UI, documents service, user management service, IAM session broker service). The company decided to create a dedicated application for each of these components and assign ownership to the respective teams.

### Architecture

![design-3-architecture](https://github.com/alexpulver/adf/assets/4362270/542655ff-5144-4824-bfbd-edcdad55de1f)

### Deployment

<img src="https://github.com/alexpulver/adf/assets/4362270/528d32dc-68f0-4a9d-98b0-b28eee0cd770" width="65%">
