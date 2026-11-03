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
| 3 | Edit | this skill | Applies the conformance edits and the policy handling of new questions |
| 4 | Verify | `bal-lib-spec-verify` | No; lists bugs, corrections, gaps and questions |
| 5 | Correct | this skill | Applies the corrections, gap texts and the policy handling of new questions |
| 6 | Gap analysis | `bal-lib-spec-gap-analysis` | No; lists improvements and restriction questions |
| 6b | Verify suspected bugs | `bal-lib-spec-verify`, steps 4 and 6 only | No; only when stage 6 handed over suspected bugs |
| 7 | Decide | this skill, with the developer | Applies the decisions |

Each stage runs in a fresh subagent, for two reasons. The review must judge the specification without knowing the implementation, which a context that never read the code guarantees better than an instruction. And each stage reads the code or the specification in depth, which would exhaust a single context on a large library. The stages share their state through `.spec-work/`, so each one builds on the findings of the previous ones and cites them instead of duplicating them.

The sibling skills are in this skill's parent directory: `../bal-lib-spec-author/SKILL.md` and so on, relative to this skill's base directory. The shared references are in `references/`, two levels up. Read `references/findings.md` before starting: it defines the findings, the policy for the specification text, and the working files every stage uses.

Work **interactively** when a developer is present: the developer confirms the outline after stage 1, may accept or reject the edits of stages 3 and 5, and answers all questions in stage 7. In **report mode**, apply the edits and the policy handling of every open question, leave the questions unanswered, and return the consolidated report.

## Running a stage

Run the stages one at a time: start a stage only after the previous one has returned, even when the tool runs subagents in the background. Before the first stage:

- Fetch the specification guide once, as `references/spec-guide.md` describes, into `.spec-work/guide.md`, and give every stage its path instead of the fetch commands.
- Create `.spec-work/toolchain.md`, where stages record facts the next stages need: the `bal` and JDK versions used, which GraalVM builds native images, and anything else that took effort to find out. Give every stage its path.

Give each skill stage (1, 2, 4, 6, 6b), as a general-purpose subagent:

- the absolute path of the stage's `SKILL.md`, to read and follow step by step, the path of the `references/` directory, `.spec-work/guide.md` and `.spec-work/toolchain.md`;
- the repository path, and that it runs in **report mode**;
- a scratch directory of its own for verification programs, and a port range of its own (for example 9100–9199 for the author, 9200–9299 for verify, 9300–9399 for gap analysis, 9400–9499 for stage 6b);
- what it may write: the author writes `docs/spec/spec.md` and `.spec-work/`; every other stage writes `.spec-work/` only;
- for the review stage: do not open the implementation (`ballerina/**/*.bal` code bodies, `native/`, `compiler-plugin/`, tests), `.spec-work/trace.md`, `claims.md`, `units.md` or `stage-*.md`, and ignore the code locations in the evidence of earlier findings;
- for stage 6b: run only steps 4 and 6 of `bal-lib-spec-verify`, on the listed suspected bugs;
- to stop every background process and container it starts, and to commit, push and file nothing;
- to return its report in the report-mode format of `findings.md`.

Run stages 3 and 5 as subagents too, with this brief:

> Apply the listed changes to `docs/spec/spec.md`. Change only the specification text; do not open the implementation. Where two changes touch the same text, merge them. Keep every link and the Contents consistent after renumbering. Apply each part of a change whose value is known; leave a part whose value is marked **needs implementation check** for a later stage. Record each change as applied (or why not) where it is listed: in `.spec-work/review.md` for edits, under the finding in `.spec-work/findings.md` for findings, and in `.spec-work/decisions.md` for policy handling. Then compile the examples and check the links as `references/verification.md` describes, and report what changed. Add a run line to `.spec-work/inventory.md`.

After each stage, save its returned report as `.spec-work/stage-<n>-<name>.md`, with `<n>` the stage number in the table above, and after each stage that changes the specification, copy it to `.spec-work/spec-after-stage-<n>.md`; the specification is often untracked, so these copies are its history.

**Resuming.** Before dispatching a stage, read the run lines in `.spec-work/inventory.md` (their format is in `findings.md`). A stage whose run line names that stage and the current commit has already run; skip it unless the developer asks to repeat it. Stage 6b is never skipped this way.

## Steps

### 1. Prepare

Confirm the repository, the specification mode (new when `docs/spec/spec.md` is absent or a stub, rewrite otherwise) and the run mode. Tell the developer the stages, that the whole run takes from about forty minutes for a small library to several hours for a large one, and when their input is needed.

### 2. Author (stage 1)

Run `bal-lib-spec-author`. **Interactive:** show the feature outline from `.spec-work/inventory.md` and the counts of findings so far, and ask the developer to confirm the outline. A changed outline means repeating the stage with the developer's outline given in the prompt; findings are kept.

### 3. Review (stage 2) and edit (stage 3)

Run `bal-lib-spec-review`. Then run the edit stage with:

- the conformance edits from `.spec-work/review.md`. **Interactive:** show them grouped by guide rule and ask which to apply; by default, all. **Report mode:** all of them.
- the policy handling, from the policy table of `findings.md`, of every question added since the specification was written that has no answer: the current behaviour stays, a question with security impact becomes _unspecified_, a non-contractual detail becomes _unspecified_ or _implementation-defined_, and for a bug whose fix is a design choice the text keeps only the property every fix shares. This includes questions that challenge an intended behaviour the author chose for a bug.

### 4. Verify (stage 4) and correct (stage 5)

Run `bal-lib-spec-verify`. Then run the correct stage with the **doc** findings it produced (corrections and gap texts, where the implementation is right), the parts of edits it completed with values from the code, and the policy handling of the questions it added, as in step 3. **Interactive:** ask which corrections to apply; by default, all. Bugs leave the specification text unchanged, since it already states the intended behaviour.

### 5. Gap analysis (stage 6) and suspected bugs (stage 6b)

Run `bal-lib-spec-gap-analysis`. It changes nothing; its improvements and restriction questions join the findings. When it handed over suspected bugs, run stage 6b on them before stage 7. A problem found in another library is a note in the report, not a finding of this one.

### 6. Decide (stage 7)

Consolidate `.spec-work/findings.md` into one view: open questions, bugs (verified first), improvements ranked by demand, and the edits, corrections and policy handling applied.

**Interactive.** Present the bugs, then ask every open question as `findings.md` describes, grouped by feature, highest impact first, and ask for a decision on each improvement (plan, reject with a reason, defer). Record every answer in `.spec-work/decisions.md`. Then apply the decisions in a subagent with the stage 3 brief and the policy table of `findings.md`: an answer of "wrong" turns a question into a bug, and the text changes to the intended behaviour; an answer of "intended" keeps the current text and may resolve an _unspecified_ marker; a chosen fix for a design-choice bug replaces the shared property with the chosen behaviour; a rejected restriction gets its reason documented.

**Report mode.** The questions stay open; their policy handling was applied in stages 3 and 5.

### 7. Report

Write the consolidated report in the report-mode format of `findings.md`, in its section order:

- **Summary**: what was produced, the counts per kind, and what blocks a release-quality specification.
- **Specification**: path and counts, counted as `findings.md` defines; for a rewrite, the old statements corrected; the rules and examples that state intended behaviour the release lacks.
- **Questions**: answered, with the decision, and open, with the policy handling applied.
- **Bugs**: every bug with its reproducer and known issue, verified first. These are the candidates for issues.
- **Improvements**: ranked by demand, with the developer's decision.
- **Edits and corrections**: applied, rejected and not applied, per stage.
- **Coverage**: per stage, what was checked in depth and what only by reading, with each stage's time.
- **Notes**: problems found in other libraries.
- **Proposed changelog entry** and **Files**.

In the top-level session, write it to `.spec-work/report.md` and summarise it to the developer with its path. When this pipeline itself runs as a subagent, return the report as text and let the caller write the file. Leave writing the changelog, committing, pushing and filing issues to the developer, and do them only when asked.
