---
name: bal-lib-spec-review
description: Review a Ballerina library specification (docs/spec/spec.md) as a document, without the implementation. Checks conformance to the library specification guide and reviews the specified behaviour as a design, raising questions. Use after a specification is written or rewritten, before a specification pull request is merged, or when asked to validate or review a specification.
---

# Review a library specification

A specification derived from an implementation inherits the implementation's design, including choices nobody made on purpose. This review reads the specification as the contract a user would rely on, and asks two things of it:

- **Conformance**: does the document follow the library specification guide? These are edits, decided by the guide.
- **Design**: is the behaviour it specifies coherent, safe and deliberate? These are questions, decided by the developer.

Read only the specification and the guide. This review judges the contract as written; comparing it with the code is the job of `bal-lib-spec-verify`. When a question can only be settled by the code, mark it **needs implementation check** and move on.

Shared references, in this plugin's `references/` directory (two levels up from this skill's base directory):

- `spec-guide.md`: where the library specification guide lives.
- `findings.md`: finding kinds, interactive and report modes, and the `.spec-work/` working files.

Work **interactively** when a developer is present, and in **report mode** when running as a subagent or when told not to stop.

## Steps

### 1. Prepare

Read `spec-guide.md` and the whole guide it points to, then `findings.md`. Open `docs/spec/spec.md`. Create `.spec-work/` as `findings.md` describes, or resume from it: read `findings.md` and `decisions.md`, continue the `F<n>` numbering, and leave `trace.md` and the code locations in earlier evidence unread, since this review judges the specification without the implementation.

Done when you can name every rule of the guide's writing rules and the subsections of its feature template.

### 2. Index the specification

Build `.spec-work/spec-index.md`, the material the later passes compare across sections:

- The outline: every section, and for each feature section which template subsections it has.
- Every defined term, with its section.
- Every configuration field: section, type, default, and the stated meaning of its special values.
- Every error type, with the conditions that produce it.
- Every diagnostic code, status code or other externally visible code, with its condition.
- Every unit and limit: timeouts, sizes, counts, and the value that means "unlimited" or "disabled".
- Every place marked _unspecified_ or _implementation-defined_.
- Every example: its section, the rules it says it illustrates, and the outcome it states.

Done when every feature section is in the outline, and every table row of the specification appears in the index.

### 3. Conformance pass

Apply each rule of the guide to each section, and record a **conformance edit** for every violation in `.spec-work/review.md`, numbered `E1`, `E2`, …: location (section and line), the guide rule, the problem, and the replacement text. For a structural edit, such as restructuring a section into the template, give a restructuring plan instead of the full text. When the replacement needs a value only the code can give, such as a default, write it with that value marked **needs implementation check**. Cover at least:

- **Structure**: the top-level sections in the guide's order; numbered headings, with the guide's exceptions; a security considerations section; no change history.
- **Feature template**: every applicable subsection present and in the template's order; configuration tables with type, default and meaning; special values stated exactly; required fields marked `Required` and optional ones `absent`. Behaviour split into topic-named subsections, which the guide allows, conforms; a missing Description, Configuration or Errors subsection, or subsections out of order, does not.
- **Normative language**: key words in capitals only where an obligation or permission needs emphasis; rules in the present tense; open behaviour marked _unspecified_ or _implementation-defined_, never left vague ("a sensible default", "may vary", "usually").
- **Content**: no copied signatures or doc comments, no tutorials or recommendations, no implementation detail (internal classes, functions, data structures).
- **Examples**: each one recognisable as informative, minimal, naming the rules it illustrates, and stating an outcome its code can produce. An example whose code cannot produce its stated outcome is an edit. An example whose stated outcome follows from its code but contradicts a rule is a design question (step 4).
- **Terminology**: defined terms used consistently, without synonyms; every term with a specific meaning defined.
- **Markdown**: links resolve to existing anchors, code fences balanced and labelled with a language, paragraphs not hard-wrapped.

Done when every guide rule has been applied to every section. Record the clean combinations too, as a compact matrix, so that "no edits" is a result rather than an omission.

### 4. Design pass

Review each feature section through each **lens** below, using the index to compare across sections. Record every concern as a finding of kind **question** in `.spec-work/findings.md`, with evidence `spec`, two or three options and a recommendation. Use kind **improvement** for a capability the specification plainly lacks. When a concern is already an earlier finding, cite it in the matrix instead of recording it again.

Check in particular every place where the specification states intended behaviour for a recorded bug: when several fixes were plausible, the chosen one is a design decision that may not have been asked, so raise it as a question.

| Lens | Ask |
|---|---|
| **Defaults** | Is each default the safe and expected choice? Does a default of `0`, `-1`, `()` or an empty array mean something a user would not guess? |
| **Consistency** | Do similar fields across features share units, the meaning of "unlimited" and "disabled", naming, and error types? Does the same situation produce the same outcome everywhere? |
| **Completeness** | Does each feature specify success, every failure, boundaries (empty, first, last, exhausted, absent) and partial outcomes? Does each failure name an error type? Is each pair of features that can be combined given an interaction? |
| **Concurrency and ordering** | What happens when two strands use the same object, when callbacks overlap, and in which order are data, errors and closure observed? |
| **Lifecycle** | What happens when an object is used before it is started, after it is stopped or closed, twice, or after its owner is gone? |
| **Failure semantics** | Does each failure fail closed? Is invalid input rejected, or silently accepted, truncated, clamped or ignored? Is a partial failure observable to the caller? |
| **Security** | Are the secure choices the defaults? Can a misconfiguration silently disable a protection? Do errors expose internal detail? Are resource limits stated? |
| **Ambiguity** | Can a rule be read two ways? Do two rules contradict, or does an example contradict a rule? Is a term used without definition? |
| **Restrictions** | Does the specification forbid or reject something without a stated reason? Ask for the reason, or for the restriction to be lifted. |
| **Surprise** | Would an experienced user of comparable libraries expect a different outcome? Name the expectation. |
| **Evolution** | Does a rule promise more than needs to be promised, making a later fix a breaking change? Should it be _unspecified_ instead? |
| **Open items** | Is each _unspecified_ or _implementation-defined_ item deliberate, and should any of them be decided now? |

Done when every feature section has been through every lens, and the matrix in `.spec-work/review.md` shows each combination as findings (new or cited) or clean.

### 5. Present

Check the findings against existing issues as `findings.md` describes.

**Interactive.** First show the conformance edits grouped by guide rule, and offer to apply them; they follow from the guide and change no behaviour. Apply the ones the developer accepts. Then present the design questions as `findings.md` describes, grouped by feature, highest impact first, and record each answer in `.spec-work/decisions.md`. An answer that changes only wording can be applied to the specification now. An answer that changes behaviour cannot be applied to the specification alone: record it as a decision to change the library, and leave the specification text as it is.

**Report mode.** Return the report of `findings.md`, using its Summary, Specification (path and counts only), Questions, Improvements, Coverage and Files sections, with a **Conformance edits** section after the summary, grouped by guide rule, each edit with its location and replacement text, and a **Needs implementation check** section listing the questions that only the code can settle.

### 6. Hand over

In report mode, the report's summary is the hand-over. Interactively, give the developer the counts of conformance edits (applied and pending), design questions (answered and open) and improvements; the decisions that require library changes; and the questions marked **needs implementation check**, as input for `bal-lib-spec-verify`.

Leave committing, pushing and filing issues to the developer, and do them only when asked.
