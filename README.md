# Application Design Framework (ADF)

## Purpose
Is your organization struggling to ship software on-time, on-budget, and in-line with expectations? The Application Design Framework (ADF) helps address these challenges by transforming abstract business requirements into concrete architectural decisions through proven guidelines and tools. Teams can accelerate innovation while building resilient, secure, and cost-effective solutions. This approach benefits executives (reduced lead times, KPI definitions), product managers (alignment on requirements, accurate ETAs), and engineers (tools for translating requirements into technical design). The result? Faster time-to-market, reduced costs, and improved project success rates.

## Definitions
Application boundary <sup>[1]</sup>:
* A **body of code** that's seen by developers as a single unit
* A **group of functionality** that business customers see as a single unit
* An **initiative** that those with the money see as a single budget

Architecture <sup>[2][3]</sup>:
* An **architecture** describes how components work together. How components communicate and interact is often the focus of architecture diagrams. 

Component <sup>[4]</sup>:
* Software **components** are things that are independently replaceable and upgradeable.

## Mindset
> Application is an ownership boundary.

> Application boundary should evolve with organizational and software changes.

## Guidelines

![](/images/adf-guidelines.svg)

**Describe use case (Sales, Marketing, Product)** to clarify the problem and define business requirements. 
* Describe business flow using [domain storytelling](https://domainstorytelling.org/quick-start-guide) and/or [event storming](https://en.wikipedia.org/wiki/Event_storming). 
* Consider writing [press release/frequently asked questions (PR/FAQ)](https://www.aboutamazon.com/news/workplace/an-insider-look-at-amazons-culture-and-processes) narrative ([example](https://www.allthingsdistributed.com/2024/11/aws-lambda-turns-10-a-rare-look-at-the-doc-that-started-it.html)) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06). 
* Identify features, bounded contexts and external dependencies. 
* Document business requirements.

**Write stories (Product, Engineering)** to scope implementation. 
* Consider the following story types <sup>[5]</sup>:
  * User Story – "As a [type of user] I [want this thing] so that [I can accomplish this goal]". Example: "As a site visitor, I want to see new content when I come to the site, so I come back more often".
  * Job Story – "When [situation], I want to [motivation], so I can [expected outcome]". Example: "When it’s dinner time tonight, I want to have pizza so I can easily feed my friends".
  * Feature-Driven Development (FDD) – "[action] the [result] [by/for/of/to] a(n) [object]". Example: "Generate a unique identifier for a transaction". 
* Map stories to features.

**Define architecture (Product, Engineering)** to address business and define technical requirements. 
* Define technical flow (e.g., load balancer &#8594; API &#8594; database) based on business flow and stories. 
* Consider the following integration dimensions <sup>[6]</sup>:
  * Service discovery (e.g., IP addresses, DNS)
  * Data format (e.g., binary, XML, JSON, protobuf, Avro)
  * Interaction type (e.g., sync, async)
  * Interaction style (e.g., messaging, RPC, query, GraphQL)
* Identify application boundaries and components. Use “fracture planes” <sup>[7]</sup> to help decide on application boundaries:
  * Profit and loss group
  * Business domain bounded context
  * Regulatory compliance
  * Change cadence
  * Team location
  * Risk
  * Performance isolation
  * Technology
  * User personas
* Document technical requirements.

**Choose technologies (Engineering)** to address technical requirements. 
* Consider building proof of concept (POC) for new technologies to validate feasibility. 
* Review decisions based on the following pillars <sup>[8]</sup>:
  * Operational excellence
  * Security
  * Reliability
  * Performance efficiency
  * Cost optimization
* Document decisions using [architectural decision records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html).
* Update stories scope and estimates based on architecture and technologies.

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
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact

## Related guidance
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done
* [AWS Decision Guides](https://aws.amazon.com/getting-started/decision-guides/) - choose the AWS services that might be right for you and your use cases

## Ongoing research
* A mechanism for introducing ADF into organization (inputs, tools, adoption, inspection, iteration, outputs).
* Organizing ADF information for multiple use cases and applications.
* Using [conditions of satisfaction (acceptance criteria)](https://www.mountaingoatsoftware.com/blog/clarifying-the-relationship-between-definition-of-done-and-conditions-of-sa) together with or instead of the requirements section.
* Linking architecrual decision records (ADRs) to requirements and stories (e.g., referencing the related requirement(s) and story(ies) in ADR context) to check impact of an architectural change during design phase.

## References
1. Martin Fowler - [ApplicationBoundary](https://martinfowler.com/bliki/ApplicationBoundary.html)
2. AWS Well-Architected Framework - [Definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html)
3. Neal Ford and Mark Richards - [Software Architecture: the Hard Parts](https://www.infoq.com/podcasts/software-architecture-hard-parts/)
4. Martin Fowler - [SoftwareComponent](https://martinfowler.com/bliki/SoftwareComponent.html)
5. Martin Fowler - [User Story](https://martinfowler.com/bliki/UserStory.html)
6. Gregor Hohpe - [The Many Facets of Coupling](https://www.enterpriseintegrationpatterns.com/ramblings/coupling_facets.html)
7. Matthew Skelton - [Designing organizations for responsiveness](https://blog.matthewskelton.net/2017/11/07/designing-organisations-for-responsiveness/#more-2053)
8. AWS Well-Architected Framework - [Pillars](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)
