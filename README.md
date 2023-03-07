# Application Design Framework (ADF)
Builders maintain and evolve applications over time. They need to do so safely (reduce blast radius) and effectively (build on existing knowledge) to reduce risk and increase velocity. ADF aims to help builders achieve these goals.

ADF uses the following [definitions](https://docs.aws.amazon.com/wellarchitected/latest/framework/definitions.html) from the AWS Well-Architected Framework (generalized):
* A **component** is the code, configuration, and resources that together deliver against a requirement. A component is often the unit of technical ownership, and is decoupled from other components.
* An **architecture** describes how components work together. How components communicate and interact is often the focus of architecture diagrams.

ADF introduces the following recommendations:
1. **Work backwards** to clarify the problem and the solution. Write use cases or [press release/frequently asked questions (PR/FAQ)](https://productstrategy.co/working-backwards-the-amazon-prfaq-for-product-innovation/) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06).
2. **Write stories** to start the conversation about requirements.
3. **Capture requirements** to guide architectural decisions. Describe flows for each story to identify requirements. Consider functional and non-functional requirements.
4. **Create architecture** to address the requirements. Identify components. Describe flows for each story on architecture level. Validate component boundaries using the following “fracture planes” (by [Matthew Skelton](https://blog.matthewskelton.net/about/)): 1/ Business Domain Bounded Context 2/ Regulatory Compliance 3/ Change Cadence 4/ Team Location 5/ Risk 6/ Performance Isolation 7/ Technology 8/ User Personas.
5. **Write [Architectural Decision Records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html)** to capture the “why” behind architectural decisions, options considered, trade-offs, and consequences for the solution and each component. ADRs allow to evolve the architecture starting from known design considerations. Consider building proof of concept (POC) to validate the decisions.
6. **Create repository and pipeline per component** to reduce blast radius and increase delivery performance.
7. **Group component infrastructure and runtime code by logical units** to align architecture with implementation. That should ease evolution and maintainability.

Related frameworks:
* [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) - apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact

Related guidance:
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done
