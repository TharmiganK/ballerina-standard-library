# Library specification guide

The library specification guide is the single source of truth for what a Ballerina library specification contains and how it is written: document roles, normative language, document structure, the feature template, writing rules, and the change process. When this plugin and the guide disagree, the guide wins.

Read the whole guide before writing or reviewing a specification. Until the proposal is merged, it is framed as a proposal; its rules are in the sections under "Description".

Try the released path first; while the proposal is unmerged it returns 404 (`base64: invalid input`), so fall back to the proposal branch:

```sh
gh api 'repos/ballerina-platform/ballerina-library/contents/docs/library-specification-guide.md' --jq .content | base64 -d \
  || gh api 'repos/ballerina-platform/ballerina-library/contents/docs/library-specification-guide.md?ref=spec-writing-guide' --jq .content | base64 -d
```

The reference migration is the HTTP library specification. Consult it for the depth expected per feature, the example density, and how rules, examples, errors and diagnostics read in practice; its content is HTTP-specific and is not a template for other libraries.

```sh
# Until ballerina-platform/module-ballerina-http#2768 is merged
gh api 'repos/TharmiganK/module-ballerina-http/contents/docs/spec/spec.md?ref=spec-refactor' --jq .content | base64 -d
# Afterwards
gh api 'repos/ballerina-platform/module-ballerina-http/contents/docs/spec/spec.md' --jq .content | base64 -d
```
