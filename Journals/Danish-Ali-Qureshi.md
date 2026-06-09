# Journal – Danish Ali Qureshi

## Week 1 (May 16–18)

- Participated in the team meeting on Friday to kickstart the project work and to define the internal deadline for completing the Design report (May 20).
- Agreed with the team to specifically work on knowledge dependencies (co-change analysis).

## Week 2 (May 19-20)

- Cloned the Express.js repository and inspected the `lib/` directory to understand the main core modules used in the Design report.
- Mined the Git history using `git log --name-only` and implemented a Python script to compute co-change frequencies between `.js` files.
- Generated the `cochange_from_git.csv` file that lists file pairs and their co-change counts.
- Selected the most relevant co-change pairs involving core `lib/` modules and router files and cross-checked them with the static dependency matrix.
- Prepared the co-change table used in Section 1.4.2 “Co-change Results”, including the “Code dependency?” classification.
- Wrote the Design report parts related to knowledge dependencies:
  - The paragraph in Section 1.1 that briefly describes the Git-based co-change analysis.
  - Section 1.4 “Knowledge Dependencies” (methodology, results, and discussion of consistency/inconsistency with static dependencies).
 
## Week 3–4 (May 21–June 1)

- Coordinated with teammates and divided the tasks.
- Reviewed the team’s partial Architecture report drafts.
- Waited for the completion of the C4 level-3 component diagrams before starting the SOLID analysis.

## Week 5 (June 2–June 9)

- Performed the SOLID analysis for the main Express core modules.
- Refactored the Architecture report (Clean Architecture, SOLID, and Architectural Characteristics sections) to bring it under the 2500-word limit.
- Integrated the design patterns (Factory, Chain of Responsibility, Decorator, Strategy) into the Design report, based on the teammate’s pattern work.
- Reworked the patterns section to match the report tone, keeping only short code snippets and stable references to the Express `lib/` structure.
- Adjusted and trimmed the Design report so it does not exceed the word limit.
