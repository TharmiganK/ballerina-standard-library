# Ballerina library skills

Agent skills for Ballerina library developers. They apply the [library specification guide](https://github.com/ballerina-platform/ballerina-library/blob/main/docs/library-specification-guide.md) to a library: write the specification from the implementation, review and verify it, and report what needs a decision.

The skills never create GitHub issues. They present findings to the developer in interactive sessions, and return a report when run as subagents.

## Skills

| Skill | Purpose | Status |
|---|---|---|
| `bal-lib-spec-author` | Write a specification from the implementation, or rewrite an existing one. Reports bugs and open questions found on the way. | Available |
| `bal-lib-spec-review` | Check a specification against the guide, and raise design questions. Does not read the implementation. | Planned |
| `bal-lib-spec-verify` | Check a specification against the implementation, and report bugs and grey areas. | Planned |
| `bal-lib-spec-gap-analysis` | Compare the library with the standards it implements and with established libraries, and report improvements and restrictions to question. | Planned |
| `bal-lib-agent-setup` | Prepare a library repository for agents. | Planned |

Shared material lives in [`references/`](references): where the guide is, the finding kinds and how the specification treats each, and how claims and examples are verified by running them.

## Install

From a local clone:

```
claude --plugin-dir /path/to/bal-lib-skills
```

Then, in a library repository, ask for the specification to be written, or invoke the skill directly with `/bal-lib-spec-author`.

## Prerequisites

- The Ballerina distribution that the library targets, for building and running verification programs.
- The library repository checked out at the code to be specified.
