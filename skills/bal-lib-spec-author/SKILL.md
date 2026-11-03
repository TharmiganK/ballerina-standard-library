---
name: bal-lib-spec-author
description: Write a Ballerina library specification (docs/spec/spec.md) from the implementation, or rewrite an existing one to follow the library specification guide. Use when a library has no specification, when its specification is outdated or written as a feature guide, or when asked to derive a specification from code.
---

# Author a library specification from the implementation

The implementation is the evidence; the specification is the contract derived from it. Every rule you write is traced to the code that implements it, and verified by running it where the toolchain allows. Deriving rules this way surfaces **findings**: bugs, and behaviour nobody decided. Those go to the developer, not into the contract.

Shared references, in this plugin's `references/` directory (two levels up from this skill's base directory):

- `spec-guide.md`: how to fetch the library specification guide, which defines the structure, the feature template and the writing rules, and the reference specification.
- `findings.md`: finding kinds, evidence, how the specification treats each finding, interactive and report modes, and the `.spec-work/` working files.
- `verification.md`: what to probe, how to verify a claim by running it, and how to compile and run examples.

## Modes

- **New**: the library has no specification, or only a stub. Write it from the implementation.
- **Rewrite**: the library has a specification. Write a new one from the implementation, and account for every statement of the old one.

Work **interactively** when a developer is present, stopping at the two checkpoints below. Work in **report mode** when running as a subagent or when told not to stop: skip the checkpoints, and return the report described in `findings.md`.

## Steps

### 1. Prepare

Read `spec-guide.md`, fetch and read the whole guide it points to, then read `findings.md` and `verification.md`. Determine the mode from `docs/spec/spec.md`. Create `.spec-work/` as `findings.md` describes, or resume from it.

Done when you can name every subsection of the feature template, and `.spec-work/inventory.md` records the library, version, commit and mode (stated to the developer in interactive mode).

### 2. Inventory the library

Build `.spec-work/inventory.md`, a map from the library's public surface to the feature sections of the specification. Collect from:

- `ballerina/**/*.bal`: every public type, record (with each field's default), class, function, method, annotation, constant, enum and configurable, including submodules under `ballerina/modules/`.
- The compiler plugin: every diagnostic code with its message and the condition that reports it. Search for where each code is reported; a code that is defined but never reported is a finding.
- Native code: the behaviour behind each external function, and where configuration values are read and converted.
- Delegated platform code: when a function wraps a JDK or runtime API, that API's behaviour is the implementation. List each delegated API.
- Tests: which behaviour is asserted, as evidence of intent.
- API docs, the package README (`ballerina/README.md` or `Module.md`), the repository README, examples and, in rewrite mode, the old specification: what users have been told.

Record each absent source explicitly (for example, "no compiler plugin", "no configurables"), so an empty section reads as checked rather than skipped.

Group the surface into features, ordered as a user meets them: for a networking library, listener, service, dispatching, binding, responses, client, security; for a function library, the inputs shared by the functions, then each operation, then errors. In rewrite mode, also extract every normative statement of the old specification into `.spec-work/old-claims.md`, one line each, with its old section.

Done when every public symbol, configurable and diagnostic code is assigned to a feature, or listed as excluded with the reason (internal, deprecated, test-only).

**Checkpoint 1 (interactive).** Show the feature outline with one line per feature, and ask the developer to confirm or reshape it before rules are derived. The outline is cheap to change now and expensive later.

### 3. Derive and verify the rules

For each feature, fill the feature template of the guide from the code, following each public entry point down to its effect, through the Ballerina, native and platform layers:

- **Configuration**: every field, its type, and its default exactly as the code sets it, with constants resolved. Find each special value (`0`, `-1`, `()`, empty arrays, negative or out-of-range values) where the field is consumed, not where it is declared, including conversions in native code (units, rounding, clamping). For a function library, the shared inputs (such as a charset or an options record) take this place.
- **Behaviour**: numbered rules covering success, every failure branch, boundaries (empty, first, last, exhausted, absent), observable ordering, and state shared between connections, requests or calls. Write processing logic, such as matching, binding and retrying, as steps.
- **Errors**: the concrete error type for each failure, and whether it is returned, logged, turned into a response, or ends the program.
- **Interactions**: how the feature combines with the others, including the order in which they apply, and properties that span operations, such as round trips.
- **Compile-time validation**: the diagnostic codes from the inventory, with their conditions.

Verify as you derive. Whenever reading leaves a rule in doubt, run it as `verification.md` describes before writing it; the probe list there names what reading gets wrong most often. When a run contradicts what the code seemed to say, the run wins: correct the rule, re-read the code to find why, and re-check the findings that relied on it.

Record the **trace** of each rule and each configuration, error and diagnostic table row in `.spec-work/trace.md`: the code locations (functions or methods rather than bare line numbers), and `verified` with the input and observed output, or `read` with the reason it was not run. The trace stays out of the specification. When code, tests, API docs or the old specification disagree, or when the behaviour looks accidental, record a finding at once, classified as `findings.md` describes. In rewrite mode, mark each old claim as kept, corrected (with its finding), moved (with the destination from the guide's document roles), or dropped (with the reason).

For a library with more than about ten feature sections, dispatch one subagent per feature group, in report mode. Give each one the paths of the three references, the guide, its slice of the inventory, a distinct port range, and the expected output: the section draft, its trace and its findings. Then merge: renumber findings into the single `F<n>` sequence, and check the interactions between sections, which no single subagent can see. A smaller library is faster in one context.

Done when every inventory item is covered by a configuration row or a rule, every rule and table row has a trace marked `verified` or `read` with a reason, every bug is verified or states why it could not be, and in rewrite mode every old claim is marked.

### 4. Present the findings

Check the findings against existing issues as `findings.md` describes.

**Checkpoint 2 (interactive).** Present the findings as `findings.md` describes. Bugs are shown with their reproducers, and questions are asked in groups by feature, including the fix choice for each bug whose intended behaviour is a design choice. Record every answer in `.spec-work/decisions.md`, then apply the decisions: an answer can turn a question into a bug, a decision or an improvement.

In report mode, apply the policy table of `findings.md` to the unanswered questions, record each default handling in `.spec-work/decisions.md`, and continue.

Done when every question has a recorded decision or default handling.

### 5. Write the specification

Write `docs/spec/spec.md` following the guide's structure and writing rules. In addition:

- Treat each finding as the policy table in `findings.md` says.
- Add an example wherever a rule is hard to follow on its own: the overall usage of a feature, precedence and matching, binding, computed values, wire formats, boundary inputs, and interactions. Each example states its outcome and the rules it illustrates. An example may show intended behaviour that the current release lacks because of a recorded bug; cite the finding in the trace.
- Collect the security-relevant behaviour and defaults into the security considerations section, stating each risk and the configuration that avoids it as facts.
- Take _Owners_ from `CODEOWNERS` or the repository's maintainers, set _Created_ and _Updated_ to today, and leave _Reviewers_ for the owners to fill, flagged in the hand-over.
- In rewrite mode, move removed content to its destination in the same change, such as static code rule explanations to their own document, or confirm that it already exists there, and list what moved where.
- Keep version history out of the specification. Changes are recorded in the library's changelog file (`CHANGELOG.md` or `changelog.md`).

### 6. Check the document

- Compile every example in the specification, `README.md` and the API docs, and assert the outcomes the specification's examples state, as `verification.md` describes.
- Resolve every internal link to an existing heading anchor, and check that code fences are balanced.
- Apply the reviewer checklist of the guide's change process to the whole document.

Done when the examples build without errors, every stated outcome holds or cites a recorded bug, no link is broken, and every checklist item holds.

### 7. Hand over

Give the developer the report sections of `findings.md`: the specification path and counts (feature sections, numbered rules, table rows, examples); findings by kind, verified first, with known issues marked; open questions; in rewrite mode, the corrected old statements, which belong in the pull request description; the rules and examples that state intended behaviour the current release lacks; the coverage; the fields left for the owners; and a proposed changelog entry.

Leave writing the changelog, committing, pushing and filing issues to the developer, and do them only when asked.
