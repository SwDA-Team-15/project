# Journal - Sina Mahdavi, 343800

## Week 1 (May 11 - May 17, 2026)

### Activities & Effort

* Reviewed the project requirements and the structure expected for the Software Architecture report.
* Looked at the Express.js GitHub repository to understand the main folders and files, especially the `lib/` directory.
* Read the already planned project sections to understand how the team wanted to organize the design and architecture reports.
* Started identifying which part of the Architecture report I could contribute to.

### Contributions to the Report

* **Report Software Architecture:** Reviewed the expected C4 architecture levels and focused on understanding the role of the Component Level diagram.
* **Report Software Architecture:** Started preparing for the C4 Level 3 work by checking how Express.js is organized internally.

---

## Week 2 (May 18 - May 24, 2026)

### Activities & Effort

* Studied the main Express.js core files, including `lib/express.js`, `lib/application.js`, `lib/request.js`, `lib/response.js`, `lib/view.js`, and `lib/utils.js`.
* Compared the source files with the team’s design analysis to understand which files were already discussed in the dependency and pattern sections.
* Started learning how C4 Component diagrams should be structured and how they differ from Context and Container diagrams.
* Investigated PlantUML and C4-PlantUML as possible tools for creating the Component Level diagram.

### Contributions to the Report

* **Report Software Architecture:** Identified the Express Core area as the most suitable target for the Component Level diagram.
* **Report Software Architecture:** Collected the initial list of internal Express responsibilities that could become components in the C4 Level 3 diagram.

---

## Week 3 (May 25 - May 31, 2026)

### Activities & Effort

* Studied the previous C4 diagrams created by the team to make sure the component diagram would be consistent with the already defined architectural view.
* Analyzed the Express Core container and separated its internal responsibilities into meaningful components instead of only listing source files.
* Mapped the main responsibilities of `lib/express.js` and `lib/application.js` to architectural components such as Public API Facade, Application Factory, Request Lifecycle Coordinator, Middleware & Route Registration API, Settings & Engine Registry, Rendering Coordinator, and HTTP Server Bootstrapper.
* Checked how the component diagram should interact with the other containers already described in the Container Level section, such as Router & Middleware Engine, Request / Response Enrichment, View Engine, Static File Middleware, and Body Parser Middleware.

### Contributions to the Report

* **Report Software Architecture:** Drafted the first version of **Section 3 (Component Level - C4 Level 3)**.
* **Report Software Architecture:** Created the first PlantUML/C4-PlantUML version of the Express.js Component Diagram.
* **Report Software Architecture:** Added the initial component explanation describing the role of each internal Express Core component.

---

## Weeks 4-5 (June 1 - June 10, 2026)

### Activities & Effort

* Refined the component diagram after feedback to make it more consistent with the Context and Container diagrams.
* Changed the diagram boundary to focus specifically on the **Express Core Container**, instead of decomposing the whole framework again.
* Moved other previously defined containers outside the Express Core boundary and represented them as external collaborators.
* Improved the component names so that they describe architectural responsibilities rather than only file names.
* Added method-call type labels under the component names, such as Public API Call, Internal Factory Call, Internal Lifecycle Call, Internal Configuration Call, Internal Render Call, Public Registration Call, Public Middleware Call, and External Runtime Call.
* Re-generated the final `component-diagram.png` from the PlantUML source and checked that it rendered correctly in the report.

### Contributions to the Report

* **Report Software Architecture:** Finalized the **Component Level (C4 Level 3)** diagram for the Express Core container.
* **Report Software Architecture:** Updated the component diagram to align with the previous Container Level structure and terminology.
* **Report Software Architecture:** Wrote and refined the component-level explanation, clarifying how Express Core creates applications, registers middleware and routes, coordinates requests, delegates rendering, and starts the HTTP server.
* **Report Software Architecture:** Saved the diagram source as `img/component-diagram.puml` and the rendered image as `img/component-diagram.png`.
