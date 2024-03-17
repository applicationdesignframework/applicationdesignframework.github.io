# Application Design Framework (ADF)
Align business with technology, minimize rework, ease evolution.

## Context
Building products and services requires companies to be responsive to market signals and demand. Some companies struggle with business agility due to misalignment between business and technology. This misalignment results in significant rework and a difficult to evolve architecture. 

Application Design Framework (ADF) aims to help address these challenges by introducing a [set of recommendations](#recommendations) and a [document template](templates/application/README.md). ADF document integrates decisions across product engineering steps in a single place to maintain alignment.

## Definitions

Application boundary <sup>[1]</sup>:
* A **body of code** that's seen by developers as a single unit
* A **group of functionality** that business customers see as a single unit
* An **initiative** that those with the money see as a single budget

Architecture <sup>[2][3]</sup>:
* An **architecture** describes how components work together. How components communicate and interact is often the focus of architecture diagrams. Architecture characteristics, often referred to as “the -ilities,” are orthogonal to the domain functionality. 

Component <sup>[4]</sup>:
* Software **components** are things that are independently replaceable and upgradeable.

## Mindset
> Application is an ownership boundary.

> Application boundary should evolve with organizational and software changes.

## Recommendations
ADF introduces the following recommendations along with related personas:
1. **Capture use cases [Sales, Marketing, Product]** to clarify problem and solution. Consider writing [press release/frequently asked questions (PR/FAQ)](https://productstrategy.co/working-backwards-the-amazon-prfaq-for-product-innovation/) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06).
2. **Write stories and flows [Product, Engineering]** to start conversation about requirements. Consider using [event storming](https://en.wikipedia.org/wiki/Event_storming).
3. **Define requirements [Product, Engineering]** to guide architectural decisions. Consider functional and non-functional requirements.
4. **Define architecture [Product, Engineering]** to address requirements. Describe stories and flows on architecture level. Identify applications. Validate application boundaries using the following “fracture planes” <sup>[5]</sup>: 1/ Profit and Loss Group 2/ Business Domain Bounded Context 3/ Regulatory Compliance 4/ Change Cadence 5/ Team Location 6/ Risk 7/ Performance Isolation 8/ Technology 9/ User Personas. Capture [Architectural Decision Records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html) including “why” behind decisions, options considered, trade-offs, and consequences. ADRs allow to evolve the architecture starting from known design considerations. Consider building proof of concept (POC) to validate the decisions.
5. **Create one or more repositories and a single pipeline per application [Engineering]** to reduce blast radius and increase delivery performance.
6. **Organize application code and resources configuration code per application component [Engineering]** to ease evolution and maintenance.

## Examples
* [High-level application design from use case to code](examples/application-design/README.md)
* [Application boundary and deployment model evolution](examples/application-evolution/README.md)
* [IAM Session Broker](examples/iam-session-broker/README.md)

## Templates
* [Application](templates/application/README.md)

## Related frameworks
* [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) - apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact

## Related guidance
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done

## References
1. Martin Fowler - [ApplicationBoundary](https://martinfowler.com/bliki/ApplicationBoundary.html)
2. AWS Well-Architected Framework - [Definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html)
3. Neal Ford and Mark Richards - [Software Architecture: the Hard Parts](https://www.infoq.com/podcasts/software-architecture-hard-parts/)
4. Martin Fowler - [SoftwareComponent](https://martinfowler.com/bliki/SoftwareComponent.html)
5. Matthew Skelton - [Designing organizations for responsiveness](https://blog.matthewskelton.net/2017/11/07/designing-organisations-for-responsiveness/#more-2053)
