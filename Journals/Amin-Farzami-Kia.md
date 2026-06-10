# Journal - Amin Farzami Kia, s338999

## Weeks 1-2 (May 11 - May 24, 2026)

### Activities & Effort

- Explored the Express.js GitHub repository to understand the overall structure of the `lib/` folder and its main modules.
- Worked with the team on setting up the dependency analysis and understanding how the framework is organized.
- Started reading into design patterns to figure out which ones were actually present in the Express source code.

### Contributions to the Report

- **Report Software Design:** Contributed to the design patterns section by identifying four patterns in the Express.js codebase: Factory, Chain of Responsibility, Decorator, and Strategy.
- **Report Software Design:** For each pattern, wrote the explanation of which classes play which role, why the pattern is used, and what the alternative would be with pros and cons.

---

## Week 3 (May 25 - May 31, 2026)

### Activities & Effort

- Reviewed the full design report together with the team to align the patterns section with the rest of the report in terms of tone and word limit.
- Started working on the architecture report, focusing on the C4 diagrams.
- Studied the C4 model and looked into available tooling to decide what to use for the diagrams.

### Contributions to the Report

- **Report Software Architecture:** Wrote the **Context Level (C4 Level 1)** section, including the diagram description, the actors, and the external systems table.
- **Report Software Architecture:** Started drafting the **Container Level (C4 Level 2)** section, producing the initial container diagram and explanations.

---

## Weeks 4-5 (June 1 - June 10, 2026)

### Activities & Effort

- Refined the container diagram multiple times to make sure it was fully aligned with the context diagram — adding the missing body-parser and router package containers, fixing external system names, and cleaning up container descriptions.
- Switched the container diagram tooling to **Structurizr DSL** and wrote the full workspace code including styles, relationships with protocols, and layout.
- Reviewed the Clean Architecture section to make sure the connection between the container structure and the Clean Architecture blueprint was clearly explained.

### Contributions to the Report

- **Report Software Architecture:** Finalized **Section 2 (Container Level)**, including the Structurizr DSL diagram, the per-container explanations, and the Clean Architecture relationship paragraph.
- **Report Software Architecture:** Ensured all container descriptions focused on what each container does rather than listing internal methods or file names.
- **Report Software Architecture:** Added protocol labels to all relationships in the container diagram (e.g. HTTP/HTTPS, Internal function call, npm require, Callback fn(path, options, cb)).
