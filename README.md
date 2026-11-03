# Ballerina library skills

Claude Code skills that apply the [library specification guide](https://github.com/ballerina-platform/ballerina-library/pull/9460) to a Ballerina library: write the specification from the implementation, review it, verify it against the code, and compare the library with standards and established libraries in other languages.

The skills never create GitHub issues, commits or pull requests. They present what they find, ask you to decide, and leave filing and committing to you.

## Skills

| Skill | Use it to |
|---|---|
| `bal-lib-spec-pipeline` | Do everything below in order and get one consolidated report. Start here. |
| `bal-lib-spec-author` | Write `docs/spec/spec.md` from the implementation, or rewrite an existing specification. |
| `bal-lib-spec-review` | Review the specification as a document: guide conformance and design questions. It does not read the code. |
| `bal-lib-spec-verify` | Check every claim of the specification against the code by reading and running it, and find behaviour the specification does not cover. |
| `bal-lib-spec-gap-analysis` | Find what the library lacks compared with the standards it implements and with established libraries, and question its restrictions. |

Shared material lives in [`references/`](references): where the guide is, the finding kinds and how the specification treats each, and how claims and examples are verified by running them.

## Install

You need [Claude Code](https://docs.claude.com/en/docs/claude-code), the Ballerina distribution your library targets (`bal`), and the GitHub CLI (`gh`, logged in).

1. Clone the `bal-lib-skills` branch to a permanent location:

   ```sh
   git clone -b bal-lib-skills https://github.com/TharmiganK/ballerina-standard-library.git ~/bal-lib-skills
   ```

2. Start Claude Code with the skills loaded, from your library's repository:

   ```sh
   claude --plugin-dir ~/bal-lib-skills
   ```

   To avoid typing the flag every time, add an alias to your shell profile (`~/.zshrc` or `~/.bashrc`):

   ```sh
   alias claude-bal='claude --plugin-dir ~/bal-lib-skills'
   ```

3. To get later changes, pull the branch: `git -C ~/bal-lib-skills pull`.

Keep the clone as it is: the skills read the shared `references/` directory next to them, so copying individual skill folders elsewhere breaks them.

## Run

Open Claude Code at the root of your library's repository, with the code you want specified checked out, and ask for what you need. Either describe it:

```
Write the specification for this library from the implementation.
Review docs/spec/spec.md before I open the pull request.
Verify the specification against the implementation.
What does this library lack compared with other libraries?
```

or invoke a skill by name, for example `/bal-lib-skills:bal-lib-spec-pipeline`.

### Interactive or report mode

- **Interactive** (the default when you are in the session): the skill stops at checkpoints. It asks you to confirm the feature outline, shows bugs with their reproducers, and asks its questions a few at a time. Your answers are applied to the specification.
- **Report mode**: say "run in report mode", or run the skill as a subagent. It does not stop; open questions stay open under the policy below, and you get one report at the end.

### What you get

- `docs/spec/spec.md`, written or updated (author and pipeline only; review, verify and gap analysis change it only with your agreement).
- `.spec-work/` at the repository root, ignored by Git through `.git/info/exclude`:
  - `findings.md`: every finding, numbered `F1`, `F2`, …: **bugs** (verified by running them, with a reproducer), **questions** (behaviour nobody decided), **improvements** (missing capabilities, ranked by demand) and **doc** corrections.
  - `decisions.md`: your answers.
  - `report.md` (pipeline): everything consolidated.
  - Working files of each skill, which let an interrupted run resume where it stopped.
- Reproducers in a scratch directory, which can become regression tests.

### How the specification treats findings

- A **bug** is written as the intended behaviour, and the bug is listed for you to fix or file. When the right fix is a design choice, you are asked.
- An open **question** is written as the current behaviour, or as _unspecified_ when it affects security, until you answer.
- An **improvement** is not written into the specification until it is implemented.

## Tips

- Start with `bal-lib-spec-pipeline` on a new or outdated specification. Run `bal-lib-spec-review` on a pull request that changes the specification, `bal-lib-spec-verify` before a release, and `bal-lib-spec-gap-analysis` when planning.
- A full pipeline run takes about an hour for a library of the size of `tcp`, longer for larger ones. Large libraries are split across subagents automatically.
- Verification runs small Ballerina programs against the released version of your library from Ballerina Central, on local ports. Libraries that talk to external systems may need Docker for a local server; behaviour that cannot be run is reported as unverified.
- A run is a sample: a second run, or a different comparison set, can find what the first one missed.
- Delete `.spec-work/` to start over.

## Feedback

Report problems and suggestions to the library team, with the skill name and the part of the report that went wrong.
