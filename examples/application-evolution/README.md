# Application boundary and deployment model evolution

This example describes application boundary and deployment model evolution as part of company growth. It doesn't go into details of stories and flows, requirements, architecture and code.

## Initial design

Company wanted to build a DocuStar product that allows customers to store, view, edit, and share documents. Engineering team decided to create a DocuStar application that contains Toolchain and Service components, each deploying as stack. Toolchain stack contains Deployment Pipeline and Pull Request Build components. Service stack includes Documents Console, Documents Store, User Management, and Credentials Broker components.

Engineering decided to create a single code repository for the DocuStar application. Application (metadata) and Toolchain resources deploy as a stack each. Service resources deploy as multiple stacks: 1/ Documents (Documents Console and Documents Store) 2/ User Management 3/ Credentials Broker.

Flow:

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/759a224d-04d6-4a0b-bc13-d55258840465">
</p>

Stacks:

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/c6201ec3-33c7-43a4-8dd8-a4cffbfb7584" width="30%">
</p>

## Design evolution

As DocuStar grew in functionality, the company increased the team working on the product. To maintain fast flow, the company decided to split DocuStar into multiple products and applications, with each application having a dedicated team:
* Public DocuStar product with Documents Console and Documents Store applications
* Internal DocuAuth product with User Management and Credentials Broker applications

Engineering decided to create a source code repository for each application. Now, Application (metadata), Toolchain, and Service resources deploy as a stack each.

Flow:

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/f53e5aa2-901e-4209-aa45-0382736a1f14">
</p>

Stacks:

<p align="center">
  <img src="https://github.com/alexpulver/adf/assets/4362270/49e77e5b-1ccd-4465-a06e-b2e53956ff00" width="50%">
</p>
