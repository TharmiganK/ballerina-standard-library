---
name: bal-lib-spec-author
description: Write a Ballerina library specification (docs/spec/spec.md) from the implementation, or rewrite an existing one to follow the library specification guide. Use when a library has no specification, when its specification is outdated or written as a feature guide, or when asked to derive a specification from code.
---

# Author a library specification from the implementation

The implementation is the evidence; the specification is the contract derived from it. Every rule you write is traced to the code that implements it, and verified by running it where the toolchain allows. Deriving rules this way surfaces **findings**: bugs, and behaviour nobody decided. Those go to the developer, not into the contract.

Shared references, in this plugin's `references/` directory (two levels up from this skill's base directory):

- `spec-guide.md`: where the library specification guide lives. The guide defines the structure, the feature template and the writing rules.
- `findings.md`: finding kinds, evidence levels, how the specification treats each finding, interactive and report modes, and the `.spec-work/` working files.
- `verification.md`: how to verify a claim by running it, and how to compile specification examples.

## Modes

- **New**: the library has no specification, or only a stub. Write it from the implementation.
- **Rewrite**: the library has a specification. Write a new one from the implementation, and account for every statement of the old one.

Work **interactively** when a developer is present, stopping at the two checkpoints below. Work in **report mode** when running as a subagent or when told not to stop: skip the checkpoints, and return the report described in `findings.md`.

## Steps

### 1. Prepare

Read `spec-guide.md` and the whole guide it points to, then `findings.md` and `verification.md`. Determine the mode from `docs/spec/spec.md`. Create `.spec-work/` as described in `findings.md`; if it already exists, resume from it.

Done when you can name every subsection of the feature template, the mode is stated to the developer, and any earlier findings and decisions are loaded.

### 2. Inventory the library

Build `.spec-work/inventory.md`, a map from the library's public surface to the feature sections of the specification. Collect from:

- `ballerina/**/*.bal`: every public type, record (with each field's default), class, function, method, annotation, constant, enum and configurable, including submodules under `ballerina/modules/`.
- The compiler plugin: every diagnostic code with its message and the condition that reports it. Search for where each code is reported; a code that is defined but never reported is a finding.
- Native code: the behaviour behind each external function, and where configuration values are read and converted.
- Tests: which behaviour is asserted, as evidence of intent.
- API docs, `README.md`, `Module.md` and, in rewrite mode, the old specification: what users have been told.

Group the surface into features, ordered as a user meets them (for a networking library: listener, service, dispatching, binding, responses, error handling, client, resiliency, security). In rewrite mode, also extract every normative statement of the old specification into `.spec-work/old-claims.md`, one line each, with its old section.

Done when every public symbol, configurable and diagnostic code is assigned to a feature, or listed as excluded with the reason (internal, deprecated, test-only).

**Checkpoint 1 (interactive).** Show the feature outline with one line per feature, and ask the developer to confirm or reshape it before rules are derived. The outline is cheap to change now and expensive later.

### 3. Derive the rules

For each feature, fill the feature template of the guide from the code. Follow each public entry point down to its effect, through the Ballerina and the native layers:

- **Configuration**: every field, its type, and its default exactly as the code sets it, with constants resolved. Find each special value (`0`, `-1`, `()`, empty arrays, negative or out-of-range values) where the field is consumed, not where it is declared, including conversions in native code (units, rounding, clamping).
- **Behaviour**: numbered rules covering success, every failure branch, boundaries (empty, first, last, exhausted, absent), observable ordering, and shared or concurrent state. Write processing logic, such as matching, binding and retrying, as steps.
- **Errors**: the concrete error type for each failure, and whether it is returned, logged or turned into a response.
- **Interactions**: how the feature combines with the others, including the order in which they apply.
- **Compile-time validation**: the diagnostic codes from the inventory, with their conditions.

Record each rule's **trace** in `.spec-work/trace.md` as `<section> rule <n> → <file:line>, …`. The trace stays out of the specification. When code, tests, API docs or the old specification disagree, or when the behaviour looks accidental, record a finding at once, classified as `findings.md` describes. In rewrite mode, mark each old claim as kept, corrected (with its finding), moved (with the destination from the guide's document roles), or dropped (with the reason).

For a library too large to hold in one context, dispatch one subagent per feature group, in report mode. Give each one the paths of the three references, the guide, its slice of the inventory, a distinct port range for verification, and the expected output: the section draft, its trace lines and its findings. Then merge: renumber findings into the single `F<n>` sequence, and check the interactions between sections, which no single subagent can see.

Done when every inventory item is covered by a configuration row or a rule, every rule has a trace, and in rewrite mode every old claim is marked.

### 4. Verify by running

Verify the rules as `verification.md` describes, starting with those that reading gets wrong most often: dispatching and matching precedence, data binding and conversion, status codes and error mapping, wire formats, headers and their exact spelling, defaults applied through native conversions, and anything involving restarts, retries or timing. Verify every bug before presenting it.

When a run contradicts a rule, the rule is wrong. Correct it, re-read the code to find why, and re-check the findings that relied on it. Mark each rule in the trace as `verified` (with the input and the observed output) or `read` (with the reason it was not run).

Done when every rule about runtime behaviour is marked `verified` or `read` with a reason, and no bug remains unverified without a stated reason.

### 5. Present the findings

**Checkpoint 2 (interactive).** Present the findings as `findings.md` describes. Bugs are shown with their verified reproducers, and questions are asked in groups by feature. Record every answer in `.spec-work/decisions.md`, then apply the decisions: an answer can turn a question into a bug, a decision or an improvement.

In report mode, apply the policy table of `findings.md` to the unanswered questions and continue.

Done when every question has a recorded decision, or, in report mode, a documented default handling.

### 6. Write the specification

Write `docs/spec/spec.md` following the guide's structure and writing rules. In addition:

- Treat each finding as the policy table in `findings.md` says: intended behaviour for bugs, current behaviour for open questions, _unspecified_ for open security questions.
- Add an example wherever a rule is hard to follow on its own: the overall usage of a feature, precedence and matching, binding, computed values, wire formats, and interactions. Each example states its outcome and the rules it illustrates.
- Collect the security-relevant behaviour and defaults into the security considerations section.
- In rewrite mode, move removed content to its destination in the same change, such as static code rule explanations to their own document, and list what moved where.
- Keep version history out of the specification. Changes are recorded in the library's `CHANGELOG.md`.

### 7. Check the document

- Compile every example as `verification.md` describes, and run the ones that state runtime outcomes.
- Resolve every internal link to an existing heading anchor, and check that code fences are balanced.
- Apply the reviewer checklist of the guide's change process to the whole document.

Done when the examples build without errors, no link is broken, and every checklist item holds.

### 8. Hand over

Give the developer:

- The specification path, and the counts of features, rules and examples.
- Findings by kind, verified first, and any questions still open.
- In rewrite mode, the old statements that were corrected, which belong in the pull request description and the changelog entry.
- The claims that remain `read`, with the reasons.
- A proposed `CHANGELOG.md` entry.

Leave committing, pushing and filing issues to the developer, and do them only when asked.
