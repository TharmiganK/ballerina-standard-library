# Findings

Shared by every `bal-lib-*` skill. A **finding** is anything the developer must know or decide: something the code does that it should not, something nobody decided, or something the library lacks. Skills record findings as they work, present them to the developer, and never file them anywhere. Creating GitHub issues is the developer's call, made after the run.

## Kinds

| Kind | Meaning | Typical source |
|---|---|---|
| **bug** | The implementation contradicts the intended behaviour: the behaviour is clearly wrong, it violates a MUST of a standard the library implements, it contradicts its own API docs or tests, or it is inconsistent with a sibling feature. | Code paths, runtime checks, standards |
| **question** | The behaviour is plausible but nobody visibly decided it, and a reasonable user could expect otherwise. The developer decides whether it is intended. | Grey areas, surprising defaults, undocumented edge cases |
| **improvement** | The behaviour is intended and correct, but the library lacks something users expect, or a standard recommends (SHOULD) something it does not do. | Standards, peer libraries |
| **doc** | The specification, API docs, README or examples are wrong or incomplete: they state something the implementation does not do, or omit behaviour users rely on. The implementation is right. | Doc comments, old specification text, README examples, behaviour the specification does not cover |

When unsure between **bug** and **question**, choose **question**. A wrong bug wastes a fix; a wrong question costs one answer.

## Evidence

Every finding carries its evidence, and the evidence states how it was established:

- **verified**: observed by running code against the released library version (see `verification.md`), with the exact input and observed output.
- **read**: derived from reading code only. State the code location (prefer the function or method over a bare line number, which goes stale) and the reasoning. A read finding is a hypothesis.
- **verified symptom, read cause**: the behaviour was observed, but its cause was not isolated. Say which part is which.
- **spec**: derived from the specification text alone, without reading or running code. Used by review skills; a spec finding concerns the contract, not the implementation.

Verify every bug, and every finding that involves runtime behaviour, when the toolchain allows it. Reading code gets precedence, conversions, shared state and framing wrong often enough that a read-only bug is not reported as a bug without saying so.

Before presenting findings, check each one against the existing issues with a read-only search of `ballerina-platform/ballerina-library`, the tracker of most libraries, across all its issues (issues filed by people often lack the module label), and of the library's own repository when it has issues enabled. Mark a finding that is already reported with the issue link instead of presenting it as new.

When a concern is already covered by an earlier finding in `.spec-work/findings.md`, cite that finding instead of recording a new one; record a new finding only for a different aspect.

Record a finding with these fields:

```markdown
### F<n>. <one-line statement of the behaviour, not of the fix>
- Kind: bug | question | improvement | doc
- Area: <feature section of the specification>
- Evidence: verified | read | verified symptom, read cause | spec — <code locations or specification lines, observed vs expected>
- Reproducer: <scratch package path, or the snippet>, or none
- Known: <existing issue link>, or none
- Impact: <who is affected and how, one or two sentences>
- Options: <for questions: two or three concrete choices, recommended one first>
- Spec handling: <how the specification text treats it, per the policy below; for a skill that does not write the specification, the proposed handling, marked not applied>
```

## How the specification treats a finding

The specification is the contract. It states what users may rely on, so:

| Finding | Specification text | Recorded as |
|---|---|---|
| **bug** whose intended behaviour is determined by the API docs, tests, a standard or a sibling feature | States the **intended** behaviour. The bug is listed for the developer. | bug |
| **bug** whose fix is a design choice (several plausible intended behaviours) | States the **current** behaviour until the developer chooses the fix. The choice is asked as a question. | bug and question |
| **question**, before an answer | States the **current** behaviour, so the document stays truthful while the question is open. | question |
| **question** with security impact (authentication, authorization, TLS, resource limits and exhaustion, input validation, canonicalisation and normalisation), before an answer | Marks the behaviour **_unspecified_**, so the specification does not promise an insecure behaviour. Security considerations state the risk and the configuration that avoids it, as facts. | question |
| **question** about a non-contractual detail, such as error message text beyond a stable prefix | Marks the detail **_unspecified_** or **_implementation-defined_**. | question |
| **question**, answered "intended" | Current behaviour stays. | decision |
| **question**, answered "wrong" | Becomes a **bug**: the text changes to the intended behaviour. | bug |
| **question**, answered "improve later" | Current behaviour stays. | improvement |
| **improvement** | Current behaviour; the specification does not describe features that do not exist. | improvement |
| **doc** | The specification follows the implementation. The wrong text is listed for correction. | doc |

Never state a known bug as specified behaviour: that turns the bug into the contract. When the specification states intended behaviour that the current release does not have, every example and rule affected cites the finding in the trace, and the report lists them, so that nobody mistakes the gap for an error in the specification.

## Interactive mode

When a developer is present:

1. Present findings at checkpoints the skill defines, not one by one as they are found.
2. Lead with a table: number, kind, one-line statement, evidence level, known issue.
3. Ask questions with the `AskUserQuestion` tool, at most four per call, grouped by feature area. Each question names the behaviour, the current result, and the options, recommended first. Bugs are shown, not asked; the developer may contest any of them.
4. Record every answer in the decisions log before acting on it.

## Report mode

When running as a subagent or without a developer, do not stop for answers. Apply the policy table with the questions unanswered, record each default handling in the decisions log, finish the work, and return one report:

```markdown
# <Skill> report: <library> <version and commit>

## Summary
<what was produced, counts per kind, and what blocks a release-quality result>

## Specification
<path; counts of feature sections, numbered rules, table rows and examples; for a rewrite, the old statements that were corrected>

## Questions for the developer
<findings of kind question, highest impact first>

## Bugs
<findings of kind bug, verified first>

## Improvements
## Doc corrections
## Coverage
<what was examined and run, and what was not, with reasons. A run is a sample: a second run may surface lower-ranked findings this one missed>

## Proposed changelog entry
## Files
<paths of the produced artefacts>
```

Skills omit the sections that do not apply to them, and may add their own.

## Working files

Keep working state in `.spec-work/` at the repository root so that an interrupted run can resume:

- `inventory.md` starts with one line per run: the skill, the library, the version in `ballerina/Ballerina.toml`, the commit, whether the working tree has uncommitted changes, the released version the run verified against, and the mode.
- `findings.md`: every finding, numbered `F1`, `F2`, … in discovery order, never renumbered.
- `decisions.md`: each answer as `F<n>: <decision> — <developer's words>`, with the date; in report mode, `F<n>: default — <policy row applied>`.
- Skill-specific files, named in each skill.

Add `.spec-work/` to `.git/info/exclude`, which ignores it locally without changing the repository. On resume, read `findings.md` and `decisions.md` first, and do not ask a recorded question again. Resume when the earlier runs recorded the same commit, even when the working tree has uncommitted changes such as a specification being written, or when the commits differ only by changes outside the library's code. Otherwise ask the developer whether to resume or start over; in report mode, move it to `.spec-work-<old commit>/` and start over.

Each skill names the working files it reads on resume. A skill that judges the specification without the implementation reads `findings.md` and `decisions.md` only, and leaves `trace.md` and the code locations in earlier evidence unread.
