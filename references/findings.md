# Findings

Shared by every `bal-lib-*` skill. A **finding** is anything the developer must know or decide: something the code does that it should not, something nobody decided, or something the library lacks. Skills record findings as they work, present them to the developer, and never file them anywhere. Creating GitHub issues is the developer's call, made after the run.

## Kinds

| Kind | Meaning | Typical source |
|---|---|---|
| **bug** | The implementation contradicts the intended behaviour: the behaviour is clearly wrong, it violates a MUST of a standard the library implements, it contradicts its own API docs or tests, or it is inconsistent with a sibling feature. | Code paths, runtime checks, standards |
| **question** | The behaviour is plausible but nobody visibly decided it, and a reasonable user could expect otherwise. The developer decides whether it is intended. | Grey areas, surprising defaults, undocumented edge cases |
| **improvement** | The behaviour is intended and correct, but the library lacks something users expect, or a standard recommends (SHOULD) something it does not do. | Standards, peer libraries |
| **doc** | The API docs, README or an existing specification state something the implementation does not do. The implementation is right. | Doc comments, old specification text |

When unsure between **bug** and **question**, choose **question**. A wrong bug wastes a fix; a wrong question costs one answer.

## Evidence

Every finding carries its evidence, and the evidence states how it was established:

- **verified**: observed by running code against the released library version (see `verification.md`), with the exact input and observed output.
- **read**: derived from reading code only. State the file and line range and the reasoning. A read finding is a hypothesis.

Verify every bug and every finding that involves runtime behaviour (status codes, headers, wire formats, binding, dispatching, timing) when the toolchain allows it. Reading code gets precedence rules, defaults conversion and framing wrong often enough that a read-only bug is not reported as a bug without saying so.

Record a finding with these fields:

```markdown
### F<n>. <one-line statement of the behaviour, not of the fix>
- Kind: bug | question | improvement | doc
- Area: <feature section of the specification>
- Evidence: verified | read — <file:line links, reproducer, observed vs expected>
- Impact: <who is affected and how, one or two sentences>
- Options: <for questions: two or three concrete choices, recommended one first>
- Spec handling: <how the specification text treats it, per the policy below>
```

## How the specification treats a finding

The specification is the contract. It states what users may rely on, so:

| Finding | Specification text | Recorded as |
|---|---|---|
| **bug** | States the **intended** behaviour. The bug is listed for the developer. | bug |
| **question**, before an answer | States the **current** behaviour, so the document stays truthful while the question is open. | question |
| **question** with security impact (authentication, authorization, input limits, TLS), before an answer | Marks the behaviour **_unspecified_**, so the specification does not promise an insecure behaviour. | question |
| **question**, answered "intended" | Current behaviour stays. | decision |
| **question**, answered "wrong" | Becomes a **bug**: the text changes to the intended behaviour. | bug |
| **question**, answered "improve later" | Current behaviour stays. | improvement |
| **improvement** | Current behaviour; the specification does not describe features that do not exist. | improvement |
| **doc** | The specification follows the implementation. The wrong text is listed for correction. | doc |

Never state a known bug as specified behaviour: that turns the bug into the contract.

## Interactive mode

When a developer is present:

1. Present findings at checkpoints the skill defines, not one by one as they are found.
2. Lead with a table: number, kind, one-line statement, evidence level.
3. Ask questions with the `AskUserQuestion` tool, at most four per call, grouped by feature area. Each question names the behaviour, the current result, and the options, recommended first. Bugs are shown, not asked; the developer may contest any of them.
4. Record every answer in the decisions log before acting on it.

## Report mode

When running as a subagent or without a developer, do not stop for answers. Apply the policy table with the questions unanswered, finish the work, and return one report:

```markdown
# <Skill> report: <library> <version or commit>

## Summary
<what was produced, counts per kind, and what blocks a release-quality result>

## Questions for the developer
<findings of kind question, highest impact first>

## Bugs
<findings of kind bug, verified first>

## Improvements
## Doc corrections
## Not verified
<claims that could not be run, and why>
## Files
<paths of the produced artefacts>
```

## Working files

Keep working state in `.spec-work/` at the repository root so that an interrupted run can resume:

- `findings.md`: every finding, numbered `F1`, `F2`, … in discovery order, never renumbered.
- `decisions.md`: each answer as `F<n>: <decision> — <developer's words>`, with the date.
- Skill-specific files, named in each skill.

Add `.spec-work/` to `.git/info/exclude`, which ignores it locally without changing the repository. On resume, read `findings.md` and `decisions.md` first, and do not ask a recorded question again.
