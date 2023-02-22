# Application Design Framework (ADF)
Builders maintain and evolve applications over time. They need to do so safely (reduce blast radius) and effectively (build on existing knowledge) to reduce risk and increase velocity.

**ADF** aims to help achieve that by introducing the following recommendations:
1. **Work backwards through use cases or [press release/frequently asked questions (PR/FAQ)](https://productstrategy.co/working-backwards-the-amazon-prfaq-for-product-innovation/) or [pitch](https://basecamp.com/shapeup/1.5-chapter-06)** to clarify the problem and the solution.
2. **Write user stories** to start the conversation about requirements.
3. **Write requirements** to guide architectural decisions. Capture UX flows. Consider user, functional, and non-functional requirements types.
4. **Design solution architecture** to address the requirements. Detail the UX flows based on the solution architecture to help identify solution components.
5. **Identify solution components** to establish boundaries and integration interfaces. Organize components using the following software boundaries or “fracture planes” (by [Matthew Skelton](https://blog.matthewskelton.net/about/)): 1/ Business Domain Bounded Context 2/ Regulatory Compliance 3/ Change Cadence 4/ Team Location 5/ Risk 6/ Performance Isolation 7/ Technology 8/ User Personas.
6. **Write [Architectural Decision Records (ADRs)](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/appendix.html)** to capture the “why” behind architectural decisions, options considered, trade-offs, and consequences for the solution and each component. ADRs allow to evolve the architecture starting from known design considerations. Add links to What’s New for services involved since the ADR publish date. New releases can impact the decision, so readers should understand the options available after the decision was made.
7. **Create repository and pipeline per component** to reduce blast radius and increase delivery performance.
8. **Group infrastructure and runtime code by function by type** to maintain the modular architecture design in the implementation to ease evolution and maintainability.
9. **Document decisions in code (infrastructure and runtime) including “why”s** to ease evolution and maintainability.

Related frameworks:
* [AWS Well-Architected Review](https://aws.amazon.com/architecture/well-architected/) - apply architectural best practices for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems.
* [Operational Readiness Review](https://docs.aws.amazon.com/wellarchitected/latest/operational-readiness-reviews/wa-operational-readiness-reviews.html) - ensure a consistent review of operational readiness, with a specific focus on eliminating known, common causes of impact

Related guidance:
* [Awesome Architecture](https://github.com/alexpulver/awesome-architecture) - concepts and foundations, followed by jobs-to-be-done
