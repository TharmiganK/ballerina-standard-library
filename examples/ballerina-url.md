# Example run: ballerina/url

A reference run of `bal-lib-spec-pipeline` on a library without a specification, kept to show what the skills produce and how long they take.

- **Library**: [ballerina/url](https://github.com/ballerina-platform/module-ballerina-url) 2.6.5, commit `10bd25e`, URL encoding and decoding (two functions, with native code wrapping `java.net.URLEncoder` and `URLDecoder`).
- **Mode**: new specification, report mode (no developer answers).
- **Date**: 2026-11-03, with `bal` 2201.14.0, verified against url 2.6.5 from Ballerina Central on the JVM and in a GraalVM native image.
- **Result**: [ballerina-platform/module-ballerina-url#533](https://github.com/ballerina-platform/module-ballerina-url/pull/533), the specification as a draft pull request.

## Stages

| # | Stage | Wall time | Tool calls |
|---|---|---|---|
| 1 | Author | 9.1 min | 31 |
| 2 | Review | 4.8 min | 17 |
| 3 | Edit | 1.9 min | 10 |
| 4 | Verify | 7.5 min | 27 |
| 5 | Correct | 2.3 min | 26 |
| 6 | Gap analysis | 7.4 min | 38 |
| 6b | Verify suspected bugs | skipped, none handed over | – |
| 7 | Consolidate | about 3 min | about 35 |

About 38 minutes and 625k subagent tokens in total. The author stage alone found 9 findings; the full pipeline found 22.

## Specification

The final specification has sections for charsets, encoding, decoding, errors and security considerations: 29 numbered rules, 6 tables, and 6 examples that compile against 2.6.5 with their stated outcomes checked by running them. Between the author's draft and the final version, the pipeline applied 16 conformance edits from the review and 1 correction from verification, and verification checked 69 claims: 52 conform (verified), 2 conform (read), 8 deviate, 7 open.

## Findings

### Bugs (verified)

- **`decode` panics on a malformed percent-encoded octet** (`"%"`, `"100%"`, `"%zz"`, …) with `IllegalArgumentException`, ending the program; `check` does not catch it. The API docs promise a `url:Error`. Cause: the native `decode` catches only `UnsupportedEncodingException`.
- **`encode` ignores the charset for U+0020, `*` and the octet `0x7E`** when the charset is not ASCII-compatible: `encode("=", "IBM037")` returns `"~"`, and `encode("ž", "UTF-16BE")` returns `"%01~"`, which does not decode back. Cause: the native code post-processes the encoder output as ASCII text.

### Questions (open)

Four with security impact:

- A malformed `%` in `decode`: return an `Error` or copy the `%` (WHATWG copies; Go and JavaScript fail).
- A character the charset cannot represent in `encode`: return an `Error`, or keep the silent `%3F` substitution that makes different values encode alike.
- Octets invalid in the charset in `decode`: return an `Error` or keep substituting U+FFFD.
- Memory exhaustion: `decode` ends the program while `encode` returns an `Error`.

Others: the fix for charsets that are not ASCII-compatible; `+` decoded as U+0020 (form decoding, unlike RFC 3986); fewer charsets in a native image although the package is marked `graalvmCompatible`; which charset aliases are accepted; the scope of the round-trip promise; whether the error detail and cause are part of the contract; a decode-only charset rejected only when a character needs encoding.

### Improvements (ranked by demand)

1. No API for `application/x-www-form-urlencoded` or query strings (demand 8; requested in ballerina-library#4808).
2. `charset` has no default, although RFC 3986 recommends UTF-8 (demand 5).
3. `encode` cannot leave `/` or other reserved characters unencoded (demand 3).
4. No URL parsing, building or resolution (demand 3; the module's scope was narrowed in ballerina-library#907).
5. No `byte[]` input or output (demand 2).

### Documentation

The API doc examples show `¶m2` instead of `&param2`, and neither the API docs nor the README say what `encode` encodes or that `decode` maps `+`.

### Outside this library

The `ballerina/http` client's form serialiser does not percent-encode keys, while its response side decodes them.

## Applied after the run

The run used a pipeline version in which report mode did not apply the policy handling of open questions, so the draft still promised the `%3F` substitution and the author's chosen fixes for both bugs. Before the pull request, these were changed by hand to what every option guarantees, with the rest _unspecified_: malformed `%` in `decode`, characters the charset cannot represent in `encode`, and `encode` with charsets that are not ASCII-compatible. The pipeline now applies this handling in its edit and correct stages.

## What the run changed in the skills

- The edit and correct stages apply the policy handling of new questions in report mode.
- The author decides per bug whether the fix is determined or a design choice.
- The review is isolated from the stage reports as well as the trace.
- The guide is fetched once, toolchain facts and issue searches are shared between stages, and the specification is snapshotted after each stage that changes it.
