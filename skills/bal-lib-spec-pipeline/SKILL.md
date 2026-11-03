---
name: bal-lib-spec-pipeline
description: Run the whole specification workflow on a Ballerina library in order (author or rewrite, review, verify, gap analysis), apply the behaviour-neutral corrections, and consolidate every bug, question and improvement into one report and one round of decisions. Use when asked to write, rewrite or refactor a library specification end to end, or to produce a library's specification together with its findings.
---

# Specification pipeline

This skill writes or rewrites `docs/spec/spec.md` and accumulates everything found on the way, by running the other `bal-lib-spec-*` skills as **stages**, in order, each in its own subagent:

| # | Stage | Skill | Changes the specification |
|---|---|---|---|
| 1 | Author | `bal-lib-spec-author` (new or rewrite mode) | Writes it |
| 2 | Review | `bal-lib-spec-review` | No; lists conformance edits and design questions |
| 3 | Edit | this skill | Applies the accepted conformance edits |
| 4 | Verify | `bal-lib-spec-verify` | No; lists bugs, corrections, gaps and questions |
| 5 | Correct | this skill | Applies the accepted corrections and gap texts |
| 6 | Gap analysis | `bal-lib-spec-gap-analysis` | No; lists improvements and restriction questions |
| 7 | Decide | this skill, with the developer | Applies the decisions |

Each stage runs in a fresh subagent, for two reasons. The review must judge the specification without knowing the implementation, which only a context that never read the code guarantees. And each stage reads the code or the specification in depth, which would exhaust a single context on a large library. The stages share their state through `.spec-work/`, so each one builds on the findings of the previous ones and cites them instead of duplicating them.

The sibling skills are in this skill's parent directory: `../bal-lib-spec-author/SKILL.md` and so on, relative to this skill's base directory. The shared references are in `references/`, two levels up. Read `references/findings.md` before starting: it defines the findings, the policy for the specification text, and the working files every stage uses.

Work **interactively** when a developer is present: the developer confirms the outline after stage 1, may accept or reject the edits of stages 3 and 5, and answers all questions in stage 7. In **report mode**, apply the behaviour-neutral edits, leave the questions unanswered under the policy of `findings.md`, and return the consolidated report.

## Running a stage

Dispatch each stage as a subagent with the general-purpose type, in the foreground, one at a time: each stage needs the previous one's output. Give it:

- the absolute path of the stage's `SKILL.md`, to read and follow step by step, and of the `references/` directory;
- the repository path, and that it runs in **report mode**;
- a scratch directory of its own for verification programs, and a port range of its own (for example 9100–9199 for the author, 9200–9299 for verify, 9300–9399 for gap analysis);
- what it may write: the author writes `docs/spec/spec.md` and `.spec-work/`; every other stage writes `.spec-work/` only;
- for the review stage: do not open the implementation (`ballerina/**/*.bal` code bodies, `native/`, `compiler-plugin/`, tests) or `.spec-work/trace.md`;
- to stop every background process and container it starts, and to commit, push and file nothing;
- to return its report in the report-mode format of `findings.md`.

Keep each returned report in `.spec-work/stage-<n>-<name>.md`. Before dispatching a stage, read `.spec-work/inventory.md`: a stage with a run line at the current commit has already run, so skip it unless the developer asks to repeat it.

## Steps

### 1. Prepare

Confirm the repository, the mode (new when `docs/spec/spec.md` is absent or a stub, rewrite otherwise) and the run mode. Tell the developer the stages, that each takes from several minutes to half an hour depending on the library's size, and when their input is needed.

### 2. Author (stage 1)

Run `bal-lib-spec-author`. **Interactive:** show the feature outline from `.spec-work/inventory.md` and the counts of findings so far, and ask the developer to confirm the outline. A changed outline means repeating the stage with the developer's outline given in the prompt; findings are kept.

### 3. Review (stage 2) and edit (stage 3)

Run `bal-lib-spec-review`. Then apply its conformance edits from `.spec-work/review.md`. **Interactive:** show the edits grouped by guide rule and ask which to apply; by default, all of them. **Report mode:** apply all of them, except those with a value marked **needs implementation check**, which wait for stage 4.

Apply the edits in a subagent given the specification, `review.md` and the list of edits to apply. It records each as applied in `review.md`, then compiles the examples and checks the links as `references/verification.md` describes.

### 4. Verify (stage 4) and correct (stage 5)

Run `bal-lib-spec-verify`. Then apply, in a subagent as in step 3, the **doc** findings it produced (corrections and gap texts, where the implementation is right) and the edits it completed with values from the code. **Interactive:** ask which to apply; by default, all of them. Bugs leave the specification text unchanged, since it already states the intended behaviour.

### 5. Gap analysis (stage 6)

Run `bal-lib-spec-gap-analysis`. It changes nothing; its improvements and restriction questions join the findings. Suspected bugs it hands to verification are verified by a short verify run limited to those findings, before stage 7.

### 6. Decide (stage 7)

Consolidate `.spec-work/findings.md` into one view: bugs (verified first), open questions, improvements ranked by demand, and the corrections and edits applied.

**Interactive.** Present the bugs, then ask every open question as `findings.md` describes, grouped by feature, highest impact first, and ask for a decision on each improvement (plan, reject with a reason, defer). Record every answer in `.spec-work/decisions.md`. Then apply the decisions to the specification in a subagent, following the policy table of `findings.md`: an answer of "wrong" turns a question into a bug, and the text changes to the intended behaviour; an answer of "intended" keeps the text and may resolve an _unspecified_ marker; a rejected restriction gets its reason documented. The subagent compiles the examples and checks the links again.

**Report mode.** Leave the questions open under the policy of `findings.md`.

### 7. Report

Write `.spec-work/report.md` in the report-mode format of `findings.md`, consolidating all stages:

- **Summary**: what was produced, the counts per kind, and what blocks a release-quality specification.
- **Specification**: path and counts; for a rewrite, the old statements corrected.
- **Bugs**: every bug with its reproducer and known issue, verified first. These are the candidates for issues.
- **Questions**: answered, with the decision, and open.
- **Improvements**: ranked by demand, with the developer's decision.
- **Edits and corrections applied**, and those rejected.
- **Coverage**: per stage, what was checked in depth and what only by reading.
- **Proposed changelog entry** and **Files**.

Return the report, or in interactive mode summarise it and give its path. Leave writing the changelog, committing, pushing and filing issues to the developer, and do them only when asked.
