# Application Design Framework (ADF)
Shorten lead time, minimize rework, ease evolution.

## Purpose
The Application Design Framework (ADF) helps companies respond quickly to market demands by reducing design-to-implementation gaps using easy-to-adopt [guidelines](#guidelines).

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
ADF introduces the following guidelines along with related personas:
1. **Capture use cases [Sales, Marketing, Product]** to clarify problem and solution. Write a paragraph or [press release/frequently asked questions (PR/FAQ)](https://productstrategy.co/working-backwards-the-amazon-prfaq-for-product-innovation/) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06).
2. **Write stories and flows [Product, Engineering]** to start conversation about requirements. Choose story syntax: 1/ User Story – “As a [type of user] I [want this thing] so that [I can accomplish this goal]” (e.g., “As a site visitor, I want to see new content when I come to the site, so I come back more often”) 2/ Job Story – “When [situation], I want to [motivation], so I can [expected outcome]” (e.g., “When it’s dinner time tonight, I want to have pizza so I can easily feed my friends”) 3/ Feature-Driven Development (FDD) – “[action] the [result] [by/for/of/to] a(n) [object]” (e.g., “Generate a unique identifier for a transaction”). Use [domain storytelling](https://domainstorytelling.org/quick-start-guide) and/or [event storming](https://en.wikipedia.org/wiki/Event_storming) to describe flows.
3. **Define requirements [Product, Engineering]** to guide architectural decisions. Group requirements into business and technical categories. Prioritize [architecturally significant requirements](https://en.wikipedia.org/wiki/Architecturally_significant_requirements): business value/risk, stakeholder concern, quality level, external dependencies, cross-cutting, first-of-a-kind, source of problems on past projects.
4. **Define architecture [Product, Engineering]** to address requirements. Identify application boundaries by describing stories and flows on technical level. Identify application components by describing requirements on technical level. Choose technologies. Use the following “fracture planes” <sup>[5]</sup> to help decide on the boundaries: 1/ Profit and Loss Group 2/ Business Domain Bounded Context 3/ Regulatory Compliance 4/ Change Cadence 5/ Team Location 6/ Risk 7/ Performance Isolation 8/ Technology 9/ User Personas. Capture [Architectural Decision Records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html) including “why” behind decisions, options considered, trade-offs, and consequences. ADRs allow to evolve the architecture starting from known design considerations. Consider building proof of concept (POC) to validate the decisions.
5. **Create one or more repositories and a single pipeline per application [Engineering]** to reduce blast radius and increase delivery performance.
6. **Structure resources configuration and business logic code by application component [Engineering]** to align design with implementation, streamline onboarding, simplify operations, and ease evolution.

## Examples
* [High-level application design from use case to code](examples/application-design/README.md)
* [Application boundary and deployment model evolution](examples/application-evolution/README.md)
* [IAM Session Broker application](examples/iam-session-broker/README.md)

## Templates
* [Application](templates/application/README.md)

## Related frameworks
* [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) - apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact

## Related guidance
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done

## Next areas of research
* Mechanism to adopt ADF for new or existing applications
* Organizing ADF information for multiple use cases and applications
* Use [conditions of satisfaction (acceptance criteria)](https://www.mountaingoatsoftware.com/blog/clarifying-the-relationship-between-definition-of-done-and-conditions-of-sa) together with or instead of the requirements section

## References
1. Martin Fowler - [ApplicationBoundary](https://martinfowler.com/bliki/ApplicationBoundary.html)
2. AWS Well-Architected Framework - [Definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html)
3. Neal Ford and Mark Richards - [Software Architecture: the Hard Parts](https://www.infoq.com/podcasts/software-architecture-hard-parts/)
4. Martin Fowler - [SoftwareComponent](https://martinfowler.com/bliki/SoftwareComponent.html)
5. Matthew Skelton - [Designing organizations for responsiveness](https://blog.matthewskelton.net/2017/11/07/designing-organisations-for-responsiveness/#more-2053)
