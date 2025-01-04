# Application Design Framework (ADF)

## Purpose
Reduce lead time by bridging product-to-engineering gaps.

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
1. **Describe use case and features (Sales, Marketing, Product)** to clarify problem and solution. Describe business flow using [domain storytelling](https://domainstorytelling.org/quick-start-guide) and/or [event storming](https://en.wikipedia.org/wiki/Event_storming). Identify bounded contexts and external dependencies. Consider writing [press release/frequently asked questions (PR/FAQ)](https://www.aboutamazon.com/news/workplace/an-insider-look-at-amazons-culture-and-processes) narrative ([example](https://www.allthingsdistributed.com/2024/11/aws-lambda-turns-10-a-rare-look-at-the-doc-that-started-it.html)) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06). Document business requirements.

2. **Define architecture (Product, Engineering)** to address business requirements. Describe technical flow (e.g., load balancer => api => database) based on business flow. Identify application boundaries and components. Use “fracture planes” <sup>[5]</sup> to help decide on application boundaries: 1/ profit and loss group 2/ business domain bounded context 3/ regulatory compliance 4/ change cadence 5/ team location 6/ risk 7/ performance isolation 8/ technology 9/ user personas. Consider the following integration dimensions: 1/ service discovery (e.g., IP addresses, DNS) 2/ data format (e.g., binary, XML, JSON, protobuf, Avro) 3/ interaction type (e.g., sync, async) 4/ interaction style (e.g., messaging, RPC, query, GraphQL). Document technical requirements.

3. **Choose technologies (Engineering)** to address technical requirements. Consider building proof of concept (POC) to validate feasibility. Use [architectural decision records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html) to explain decisions, options considered, trade-offs and consequences. Review decisions based on the following pillars <sup>[6]</sup>: 1/ operational excellence 2/ security 3/ reliability 4/ performance efficiency 5/ cost optimization.

4. **Write stories (Product, Engineering)** to scope implementation. Use the following story types: 1/ User Story – “As a [type of user] I [want this thing] so that [I can accomplish this goal]” (e.g., “As a site visitor, I want to see new content when I come to the site, so I come back more often”) 2/ Job Story – “When [situation], I want to [motivation], so I can [expected outcome]” (e.g., “When it’s dinner time tonight, I want to have pizza so I can easily feed my friends”) 3/ Feature-Driven Development (FDD) – “[action] the [result] [by/for/of/to] a(n) [object]” (e.g., “Generate a unique identifier for a transaction”). Align stories with features.

5. **Create one or more repositories and a single pipeline per application (Engineering)** to reduce blast radius and increase delivery performance.

6. **Organize resources configuration and business logic code by application components (Engineering)** to align architecture with code.

## Examples
* [High-level application design from use case to code](examples/application-design/README.md)
* [Application boundary and deployment model evolution](examples/application-evolution/README.md)
* [IAM Session Broker application](examples/iam-session-broker/README.md)

## Templates
* [Standard](templates/standard/README.md)

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
5. Matthew Skelton - [Designing organizations for responsiveness](https://blog.matthewskelton.net/2017/11/07/designing-organisations-for-responsiveness/#more-2053)
6. AWS Well-Architected Framework - [Pillars](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)
