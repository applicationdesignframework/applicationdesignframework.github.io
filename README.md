# Application Design Framework (ADF)
Builders maintain and evolve products over time. They need to do so safely (reduce blast radius) and effectively (build on existing knowledge) to reduce risk and increase velocity.

**ADF** aims to help achieve that by introducing the following recommendations:
1. **Identify application components** to establish component boundaries and integration interfaces. Split the application into components using the following software boundaries or “fracture planes” (by [Matthew Skelton](https://blog.matthewskelton.net/about/)): 1/ Business Domain Bounded Context 2/ Regulatory Compliance 3/ Change Cadence 4/ Team Location 5/ Risk 6/ Performance Isolation 7/ Technology 8/ User Personas.
2. **Create repository and pipeline per component** to reduce blast radius and increase delivery performance.
3. **Write [Architectural Decision Records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html)** to capture the “why” behind architectural decisions, options considered, trade-offs, and consequences. ADRs allow to evolve the architecture starting from known design considerations. Add links to What’s New for services involved since the ADR publish date. New releases can impact the decision, so readers should understand the options available after the decision was made.
4. **Perform [AWS Well-Architected Review](https://aws.amazon.com/architecture/well-architected/)** to apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
5. **Perform [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html)** to ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact.
6. **Group infrastructure and runtime code by function by type** to maintain the modular architecture design in the implementation to ease evolution and maintainability.
7. **Document decisions in code (infrastructure and runtime) including “why”s** to ease evolution and maintainability.
