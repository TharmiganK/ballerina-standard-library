---
name: bal-lib-spec-gap-analysis
description: Compare a Ballerina library with the standards it implements and with established libraries in other languages, feature area by feature area. Reports missing capabilities as improvements or features with evidence and a demand score, and questions the design reason for each restriction the specification imposes. Use when planning a library's roadmap, before a major release, or when asked what a library lacks compared with others.
---

# Gap analysis against standards and established libraries

Bugs are about what a library does wrong; gaps are about what it cannot do. This skill compares each feature area with the standards the library implements and with mature libraries in other languages, and reports what is missing or needlessly hard, with evidence the developer can check. It also asks why the specification forbids things that comparable libraries allow.

A gap is reported only when all of these hold:

- **Absent or unreasonably hard here**: not offered in any form, including another name, a configuration field, an extension point, or a short documented workaround. Read the specification and the code before calling something missing.
- **Established elsewhere**: a standard the library implements requires or recommends it, or at least two comparison libraries offer it. Name the concrete API or requirement in each.
- **Fits the library**: it makes sense for Ballerina's model. Proposing to copy an API whose idea already exists here in another form is not a gap.

Few strong gaps are worth more than many weak ones. Reporting nothing for an area is a valid outcome.

Shared references, in this plugin's `references/` directory (two levels up from this skill's base directory):

- `findings.md`: finding kinds, evidence, interactive and report modes, the existing-issue check, and the `.spec-work/` working files.
- `verification.md`: how to run a sample, when a gap concerns runtime behaviour.

Work **interactively** when a developer is present, and in **report mode** when running as a subagent or when told not to stop.

## Steps

### 1. Prepare

Read `findings.md` and `verification.md`, then the library's specification (`docs/spec/spec.md`) and public API (`ballerina/**/*.bal`). Without a specification, work from the public API, the API docs and the README, and say so in the report. Create `.spec-work/` as `findings.md` describes, or resume from it and continue the `F<n>` numbering. Earlier improvements that lack the fields of step 5 are completed in place, not recorded again.

Divide the library into **units** (feature areas). Reuse `.spec-work/units.md` when an earlier skill wrote it, or the repository's own catalogue of feature areas when it has one; otherwise derive the units from the specification's feature sections.

Done when every feature section belongs to a unit, and `.spec-work/inventory.md` records the run.

### 2. Choose the comparison set

For each unit, record in `.spec-work/references.json`:

```json
{
  "default": {
    "libraries": ["<library A (language)>", "<library B (language)>", "<library C (language)>"]
  },
  "<unit>": {
    "standards": ["<standard> section <n>"],
    "libraries": ["+<library added for this unit>", "-<library removed for this unit>"],
    "topics": ["<capability>", "<capability>"]
  }
}
```

- **Standards**: the specifications the unit implements or interoperates with (RFCs, W3C, ISO, vendor protocols), down to the relevant sections where possible.
- **Libraries**: one default set of three to five established, widely used libraries for the same purpose in other languages, preferring the standard library or the de facto choice of each language, with per-unit additions and removals. Sibling Ballerina libraries and the library's own implementation dependencies (such as the networking framework it wraps) do not count toward "established"; cite sibling libraries as evidence of fit and demand instead. When a dependency is cited, cite the version the library uses.
- **Topics**: capabilities these libraries commonly cover for the unit. The topics are where to start looking, not a list to report; add topics as the comparison reveals them.

When the repository has such a catalogue, for example a gap scanner's `references.json`, start from it.

**Checkpoint (interactive).** Show the comparison set and ask the developer to confirm or change it. The comparison set decides what counts as established.

Done when every unit has standards (or the statement that none apply), at least three libraries, and topics.

### 3. Map what the library offers

For each unit, write down in `.spec-work/gaps.md` what a user can configure and call today, from the specification and the public API, and how each topic is served: directly, through another feature, through an extension point, or with a workaround. This map becomes the coverage table of step 8. Read the code to confirm a workaround; a workaround that works but is not documented counts as hard, not as present. Run a sample for every topic whose answer concerns behaviour rather than the shape of the API.

### 4. Compare with the standards

For each standard, list the requirements that apply to the unit (MUST, SHOULD, MAY, or the standard's equivalent) and check each one against the map:

- A **SHOULD** or a widely implemented **MAY** that the library lacks is a gap.
- A **MUST** that the library itself appears to violate, in behaviour it performs, is not a gap but a suspected bug. Record it as a finding of kind **bug** with evidence `read` or `spec`, and mark it **for bal-lib-spec-verify**, which reproduces bugs. A MUST the library cannot meet by design is a question.
- A requirement that binds the implementation the library wraps (for example, the operating system's protocol stack) and that the library hides from its users is an improvement; cite the requirement level as demand. A MUST met only by a fixed default the user cannot change is recorded the same way.

### 5. Compare with the libraries

For each topic, establish what each comparison library offers, from its official documentation: cite the page and quote the phrase relied on (an anchor is optional). Use your own knowledge to choose where to look, never as the evidence: a claim about another library without a citation is not used. Keep each candidate that meets the three conditions at the top of this skill; record the ones judged not to fit, with the reason, in the coverage.

Record a kept candidate as a finding of kind **improvement**, with these fields in addition to those in `findings.md`:

```markdown
- Scope: option (an existing feature lacks an option or behaviour) | capability (a new capability)
- Today: <what the library offers, the best workaround and why it falls short, with specification sections and code locations, and the sample run>
- Elsewhere: <one line per standard or library: API or requirement, behaviour, citation with the quoted phrase>
- Demand: <score 0–10 and its evidence>
- Proposal: <a sketch that fits the library: configuration fields or function signatures in a ballerina block, compatibility, specification impact>
```

### 6. Question the restrictions

List every restriction the specification states or the implementation enforces: inputs it rejects, combinations it forbids, types it narrows, limits it fixes, compile-time diagnostics that reject code. Start from the restrictions an earlier review or earlier questions already raised, and list them as covered. Check that the implementation enforces each remaining restriction before questioning it; one it does not enforce is a **doc** finding. For each enforced restriction that no applicable standard requires and at least two comparison libraries do not impose, record a finding of kind **question**: what is restricted, who it affects, how the others behave (cited), and the options, which are to document the reason for the restriction in the specification, or to lift it. A restriction with a stated reason that still holds is not a finding.

### 7. Deduplicate and score

Check every finding against existing issues as `findings.md` describes. A candidate that an open issue already asks for is not recorded again: it stays in the ranking and the coverage, marked "reported #n", since it is often among the most wanted. A candidate that a closed issue rejected is dropped, with the issue named in the coverage. Then score each improvement's **demand** from 0 to 10 with this rubric, and state the evidence:

- +3 for an open issue in this library asking for it, +1 more per further issue or substantial discussion;
- +2 for an issue in a sibling Ballerina library asking for the same capability;
- +2 when a standard the library implements requires (MUST) or recommends (SHOULD) it, +1 for MAY;
- +1 for each comparison library that offers it on by default, up to +3.

Done when every finding has been checked against existing issues, and every improvement, including the reported ones, has a score with evidence.

### 8. Present

**Interactive.** Show the coverage table first: per unit and topic, the verdict (present, present with a workaround, gap, gap not recorded with the reason, restriction questioned, already reported, not applicable). Then present the improvements ranked by demand, and ask about each: plan it, reject it (with the reason, which may belong in the specification), or defer it. Ask the restriction questions as `findings.md` describes. Record every answer in `.spec-work/decisions.md`. Hand the suspected bugs to the developer as input for `bal-lib-spec-verify`.

**Report mode.** Return the report of `findings.md`, using its Summary, Questions, Improvements, Doc corrections, Coverage and Files sections. Order the improvements by demand, put the coverage table in the Coverage section, and add a **For bal-lib-spec-verify** section with the suspected bugs.

### 9. Hand over

In report mode, the report's summary is the hand-over. Interactively, give the developer the improvements planned, rejected and deferred, the restrictions to document or lift, the suspected bugs for verification, and the coverage. A run is a sample: another run, or another comparison set, can surface gaps this one did not.

Leave committing, pushing and filing issues to the developer, and do them only when asked.
