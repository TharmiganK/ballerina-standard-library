---
name: bal-lib-spec-verify
description: Verify a Ballerina library specification (docs/spec/spec.md) against the implementation, claim by claim, by reading the code and running it. Reports bugs where the implementation deviates, specification corrections where the text is wrong, behaviour the specification does not cover, and questions for grey areas. Use before a release, after behaviour changes, when a specification was written by hand, or when asked whether the implementation conforms to its specification.
---

# Verify a specification against the implementation

The specification is the contract; the implementation either conforms to it or does not. This skill checks every **claim** of the specification against the code, by reading it and by running it, and also walks the code to find behaviour the specification does not mention. Each deviation is resolved into a bug (the implementation is wrong), a correction (the specification is wrong), or a question (nobody can tell yet).

A bug is reported as verified only with a **reproducer** that asserts the specified behaviour and fails against the current release. A deviation that cannot be reproduced stays a hypothesis and is reported as such.

Shared references, in this plugin's `references/` directory (two levels up from this skill's base directory):

- `spec-guide.md`: how to fetch the library specification guide.
- `findings.md`: finding kinds, evidence, how the specification treats each finding, interactive and report modes, and the `.spec-work/` working files.
- `verification.md`: what to probe, how to run a claim, and how to compile and run examples.

Work **interactively** when a developer is present, and in **report mode** when running as a subagent or when told not to stop.

## Steps

### 1. Prepare

Read `spec-guide.md`, fetch the guide and read its feature template and writing rules, then read `findings.md`, `verification.md` and the whole specification. Create `.spec-work/` as `findings.md` describes, or resume from it and continue the `F<n>` numbering. Questions and conformance edits that an earlier review marked **needs implementation check** (in `.spec-work/findings.md` and `.spec-work/review.md`) are input to step 4: settle each one from the code, and record the answer or the missing value.

Done when `.spec-work/inventory.md` records the library, version, commit and mode, and you know which released version the checks run against.

### 2. Divide into units

A **unit** is a feature area that can be verified on its own: one or more specification sections, the code paths that implement them (Ballerina, native and compiler plugin), and the test packages or files that exercise them. Record the units in `.spec-work/units.md`. When the repository already has a catalogue of feature areas, such as a scanner's scope file, reuse its division.

Done when every section of the specification belongs to exactly one unit, and every unit names its code paths.

### 3. List the claims

For each unit, list every checkable **claim** in `.spec-work/claims.md`, each with an identifier (`<section>/r<n>` for a rule, `<section>/<field>` for a table row, `<section>/ex<n>` for an example):

- every numbered rule, split into separate claims where one rule states several things;
- every configuration row: the type, the default and each stated special value;
- every error row and diagnostic row: the condition and the resulting error or code;
- every interaction;
- every outcome an example states;
- every _unspecified_ or _implementation-defined_ marker, as a claim that the specification deliberately leaves it open.

Done when every rule, table row and example of the specification has at least one claim.

### 4. Check each claim

Check the claims unit by unit. Trace each claim to the code that implements it, then run it as `verification.md` describes whenever it concerns runtime behaviour; read-only checks suffice for types, defaults that are used unconverted, and diagnostics confirmed by a build. Record one outcome per claim in `claims.md`:

| Outcome | Meaning |
|---|---|
| **conforms (verified)** | The run shows the specified behaviour. |
| **conforms (read)** | The code implements the specified behaviour; not run, with the reason. |
| **deviates** | The implementation does something else. Record the observed behaviour. |
| **ambiguous** | The claim can be read in more than one way, and the implementation matches only some readings. |
| **untestable** | Neither reading nor running can settle it in this session; say what would. |

Resolve each deviation into a finding, using the evidence of intent in this order: the API docs, the tests, the standards the library implements, the behaviour of sibling features, the changelog and design proposals.

- The evidence says the implementation is wrong: a **bug**. Write a reproducer that asserts the specified behaviour, and confirm it fails. If it does not fail, the claim conforms after all: re-read the specification and the code.
- The evidence says the specification is wrong: a **doc** finding, with the corrected specification text.
- The evidence is silent or split: a **question**, with the specification text and the observed behaviour as the options.

Resolve each ambiguous claim into a **question** with the readings as options, unless the code makes one reading clearly intended, in which case it is a **doc** finding with the clarified text.

For a large specification, dispatch one subagent per unit, in report mode, each with the paths of the references, its unit, its claims, a distinct port range, and the expected output: the claims with outcomes, and its findings. Then merge, renumbering findings into the single `F<n>` sequence. A small specification is faster in one context.

Done when every claim has an outcome, every deviation is resolved into a finding, and every bug has a failing reproducer or states why none could be built.

### 5. Find what the specification does not cover

Walk each unit's code in the other direction, from the implementation to the specification:

- every public symbol, configuration field, error type and diagnostic code of the unit;
- every failure branch, and every place a value is clamped, defaulted, ignored or converted;
- every behaviour the tests assert.

Each one the specification does not state, and does not deliberately leave open, is a **gap**: record a **doc** finding with the proposed specification text, or a **question** when the behaviour looks accidental and should not become part of the contract without a decision.

Done when every public symbol, error type and diagnostic code of every unit is either referenced by the specification or recorded as a gap.

### 6. Present

Check the findings against existing issues as `findings.md` describes.

**Interactive.** Show the conformance summary first: per unit, the number of claims that conform (verified and read), deviate, are ambiguous or untestable. Then present the bugs with their reproducers, and ask the questions as `findings.md` describes, recording every answer in `.spec-work/decisions.md`. Offer to apply the doc findings to the specification, since in those the implementation is right; apply the ones the developer accepts. Leave the specification text of a bug unchanged: it already states the intended behaviour.

**Report mode.** Return the report of `findings.md`, with a **Conformance** section after the summary holding the per-unit table, and the untestable claims in the coverage section.

### 7. Hand over

Give the developer the conformance table, the findings by kind with known issues marked, the reproducers of the bugs (each is a candidate regression test, as `verification.md` describes), the specification corrections applied and pending, and the coverage. A run is a sample; say which units were checked in depth and which only by reading.

Leave committing, pushing and filing issues to the developer, and do them only when asked.
