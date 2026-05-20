# Journal – Danish Ali Qureshi

## Week 1 (May 16–18)

- Participated in the team meeting on Friday to kickstart the project work and to define the internal deadline for completing the Design report (May 20).
- Agreed with the team to specifically work on knowledge dependencies (co-change analysis).

## Week 2 (May 19)

- Cloned the Express.js repository and inspected the `lib/` directory to understand the main core modules used in the Design report.
- Mined the Git history using `git log --name-only` and implemented a Python script to compute co-change frequencies between `.js` files.
- Generated the `cochange_from_git.csv` file that lists file pairs and their co-change counts.

## Week 3 (May 20)

- Selected the most relevant co-change pairs involving core `lib/` modules and router files and cross-checked them with the static dependency matrix.
- Prepared the co-change table used in Section 1.4.2 “Co-change Results”, including the “Code dependency?” classification.
- Wrote the Design report parts related to knowledge dependencies:
  - The paragraph in Section 1.1 that briefly describes the Git-based co-change analysis.
  - Section 1.4 “Knowledge Dependencies” (methodology, results, and discussion of consistency/inconsistency with static dependencies).
