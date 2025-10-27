# Application Design Framework (ADF)

## Introduction
While individual use cases are straightforward to design in isolation, their combination often exposes conflicting architectural requirements. Design decisions that optimize one capability can compromise another, creating complex interdependencies across the system. As this complexity grows, the design phase stretches into weeks, and implementation incurs further delays due to non–value-added rework. A key driver of these delays is teams losing alignment between use cases and architecture.

The Application Design Framework (ADF) helps maintain tight alignment between use cases and architecture. Using ADF’s proven, prescriptive guidelines, you can shorten the design phase from weeks to days and avoid non–value-added rework. The result? You’ll ship the right solution faster.

## Tenets
1. **Use case-driven.** Every decision should be guided by the use case. If it doesn't serve one, it's out of scope.
2. **Traceability matters.** Every use case should map to architecture, and architecture to code. Anyone should be able to follow the thread.
3. **Architecture first, technology second.** Architecture satisfies business and technical requirements; technology implements them. Separate architecture and technology decisions to reduce cognitive load.

## Definitions
Application boundary <sup>[1]</sup>:
* A **body of code** that's seen by developers as a single unit.
* A **group of functionality** that business customers see as a single unit.
* An **initiative** that those with the money see as a single budget.

Architecture <sup>[2][3]</sup>:
* An **architecture** describes how components work together. How components communicate and interact is often the focus of architecture diagrams. 

Component <sup>[4]</sup>:
* Software **components** are things that are independently replaceable and upgradeable.

## Mindset
> Application is an ownership boundary.

> Application boundary may change over time.

## Guidelines

![](/images/adf-guidelines.svg)

**Describe use case (Sales, Marketing, Product)** to clarify the problem and define business requirements. 
* Answer the 5 customer questions <sup>[5]</sup>: 1/ who is the customer and what insights do we have about them? 2/ what is the prevailing customer problem/opportunity? what data informed this? 3/ what is the solution? why is it the right solution to address the customer need versus other alternatives? 4/ how would we describe the end-to-end customer experience (business flows)? what is the most important customer benefit? 5/ how will we define and measure success?
* Describe business flows using [domain storytelling](https://domainstorytelling.org/quick-start-guide), [story maps](https://www.mountaingoatsoftware.com/blog/user-story-mapping-how-to-create-story-maps), or [event storming](https://en.wikipedia.org/wiki/Event_storming).
* Consider writing [press release/frequently asked questions (PR/FAQ)](https://www.aboutamazon.com/news/workplace/an-insider-look-at-amazons-culture-and-processes) narrative ([example](https://www.allthingsdistributed.com/2024/11/aws-lambda-turns-10-a-rare-look-at-the-doc-that-started-it.html)) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06).
* Identify [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html) and external dependencies.
* Document business requirements using [EARS](https://alistairmavin.com/ears/) patterns: "while [optional pre-condition], when [optional trigger], the [system name] shall [system response]" (e.g., "while the user is signed-in, when the user asks to change the password, the application shall re-authenticate the user").

**Define architecture (Product, Engineering)** to address business and define technical requirements. 
* Define technical flows (e.g., load balancer &#8594; API &#8594; database) based on business flows. 
* Consider the following integration dimensions <sup>[7]</sup>: 1/ service discovery (e.g., IP addresses, DNS) 2/ data format (e.g., binary, XML, JSON, protobuf, Avro) 3/ interaction type (e.g., sync, async) 4/ interaction style (e.g., messaging, RPC, query, GraphQL).
* Identify application boundaries and components. Use “fracture planes” <sup>[8]</sup> to help decide on application boundaries: 1/ profit and loss group 2/ business domain bounded context 3/ regulatory compliance 4/ change cadence 5/ team location 6/ risk 7/ performance isolation 8/ technology 9/ user personas.
* Review decisions based on the following pillars <sup>[9]</sup>: 1/ operational excellence 2/ security 3/ reliability 4/ performance efficiency 5/ cost optimization.
* Document decisions using [architectural decision records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html).
* Document technical requirements using [EARS](https://alistairmavin.com/ears/) patterns: "while [optional pre-condition], when [optional trigger], the [system name] shall [system response]" (e.g., "while the deployment pipeline is running, when there is a change to the pipeline structure, the pipeline shall stop and restart with the new structure").

**Choose technologies (Engineering)** to address technical requirements. 
* Consider building proof of concept (POC) for new technologies to validate feasibility. 
* Review decisions based on the following pillars <sup>[9]</sup>: 1/ operational excellence 2/ security 3/ reliability 4/ performance efficiency 5/ cost optimization.
* Document decisions using [architectural decision records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html).

**Write code (Engineering)** to implement business and technical requirements.
* Create one or more repositories and a single pipeline per application to reduce blast radius and increase delivery performance.
* Organize resources configuration and business logic code by application components to align architecture with code.

## Examples
* [High-level application design from use case to code](examples/application-design/README.md)
* [Application boundary and deployment model evolution](examples/application-evolution/README.md)
* [IAM Session Broker application](examples/iam-session-broker/README.md)

## Templates
* [Standard (Markdown)](templates/Standard.txt)
* [Standard (Word)](templates/Standard.docx)

## Related frameworks
* [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) - apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
* [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework) - recommendations to help architects, developers, administrators, and other cloud practitioners design and operate a cloud topology that's secure, efficient, resilient, high-performing, and cost-effective.
* [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/) - a set of quality-driven tenets, architectural decision points, and review tools intended to help solution architects build a technical foundation for their workloads.
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact.

## Related guidance
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done.
* [AWS Decision Guides](https://aws.amazon.com/getting-started/decision-guides/) - choose AWS services that might be right for you and your use cases.
* [Azure Decision Guides](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/decision-guides/) - choose Azure services that might be right for you and your use cases.

## Ongoing research
* Backlog planning - from use cases and architecture to code:
  * Stories break down into tasks. Requirements are inputs to tasks. [Conditions of satisfaction (acceptance criteria)](https://www.mountaingoatsoftware.com/blog/clarifying-the-relationship-between-definition-of-done-and-conditions-of-sa) are the tasks outputs. Map flows to stories and requirements to tasks for traceability.
  * Consider the following story types <sup>[6]</sup>: 1/ user story – "as a [type of user] I [want this thing] so that [I can accomplish this goal]" (e.g., "as a site visitor, I want to see new content when I come to the site, so I come back more often") 2/ job story – "when [situation], I want to [motivation], so I can [expected outcome]" (e.g., "when it’s dinner time tonight, I want to have pizza so I can easily feed my friends" 3/ feature-driven development – "[action] the [result] [by/for/of/to] a(n) [object]" (e.g., "generate a unique identifier for a transaction").
* A mechanism for introducing ADF into organization (inputs, tools, adoption, inspection, iteration, outputs).
* Organizing ADF information for multiple use cases and applications.

## References
1. Martin Fowler - [ApplicationBoundary](https://martinfowler.com/bliki/ApplicationBoundary.html)
2. AWS Well-Architected Framework - [Definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html)
3. Neal Ford and Mark Richards - [Software Architecture: the Hard Parts](https://www.infoq.com/podcasts/software-architecture-hard-parts/)
4. Martin Fowler - [SoftwareComponent](https://martinfowler.com/bliki/SoftwareComponent.html)
5. Giulia Rossi and Dave Brown - [Working backwards: Amazon’s approach to innovation](https://d1.awsstatic.com/events/reinvent/2019/REPEAT_1_Working_backwards_Amazon%E2%80%99s_approach_to_innovation_ENT207-R1.pdf)
6. Martin Fowler - [User Story](https://martinfowler.com/bliki/UserStory.html)
7. Gregor Hohpe - [The Many Facets of Coupling](https://www.enterpriseintegrationpatterns.com/ramblings/coupling_facets.html)
8. Matthew Skelton and Manuel Pais - [Team Topologies](https://teamtopologies.com/key-concepts)
9. AWS Well-Architected Framework - [Pillars](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)
