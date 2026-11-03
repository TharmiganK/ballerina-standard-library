# Verification

How `bal-lib-*` skills turn a claim about a library into **verified** evidence (see `findings.md`): run a small program against the library and observe the result. A scratch package is the unit of verification. It takes seconds to build, where a repository test suite can take tens of minutes.

## Which version to run

Run against the released version that matches the code you read:

1. Read the version in `ballerina/Ballerina.toml` and the latest release tag (`git tag --sort=-v:refname | head`).
2. If the working tree is the released code, or differs only in documentation, depend on that version from Ballerina Central.
3. If the code you read contains unreleased behaviour changes, say so in the evidence, and either run against the release (stating the difference) or build the library locally and point the scratch package at the local build. The repository's own build instructions say how; `-PdeleteDependencies` or an equivalent flag is often needed, otherwise the build silently resolves the released version from Central.

## Scratch package

Create each scratch package in its own new directory outside the repository, in the session's scratchpad when one exists:

```
<scratch>/<topic>/
  Ballerina.toml     [package] org = "t", name = "<topic>", version = "0.1.0"
  main.bal
```

- Unused imports are compile errors, so import only what the file uses.
- `bal build` runs the library's compiler plugin, so a successful build also checks compile-time diagnostics. Run `bal build` and read every `ERROR` before running anything.
- Run the built jar with `java -jar target/bin/<name>.jar`, which avoids rebuilding.

## Services and clients

For libraries that open listeners:

- Declare one listener per port and attach every service to it: `listener http:Listener ep = new (9101);`. Two `new http:Listener(9101)` in one program fail with "Address already in use".
- Give each scratch package its own port range, so parallel runs do not collide.
- `main` runs before module-level listeners start, so a client in `main` cannot call a service in the same program. Run the service jar in the background, wait until it answers, and then run the client: either `curl`, or the same jar in client mode guarded by a configurable (`configurable boolean clientMode = false;` and `-CclientMode=true`). The client-mode process also tries to start the listener after `main` returns, and its "Address already in use" error is expected.
- Wait for readiness by polling (`for i in $(seq 1 30); do curl -s -o /dev/null localhost:<port>/ && break; sleep 1; done`) rather than sleeping a fixed time.
- Observe with `curl -s -i`, and drop volatile headers such as `date` and `server` before comparing. Use `od -c` when line endings or framing matter.
- Stop the background process when done (`pkill -f <name>.jar`).

## Recording the result

Copy the exact request and the relevant part of the response into the finding or the trace. A claim is verified only when the observed output matches what the claim states. When it does not, the claim is wrong, not the run: fix the claim, and re-read the code to understand why.

## When a claim cannot be run

Some behaviour needs infrastructure the session lacks: a remote service, a database, a broker, a specific operating system, or a timing window that is impractical to hit. Record the claim as **read**, say why it was not run, and list it under "Not verified" in the report. When the behaviour depends on a runtime conversion, such as `decimal` to `int`, inspect the runtime method instead of assuming it, e.g. `javap -c` on the class in the distribution's `bre/lib` jars.

## Compiling specification examples

Every Ballerina example in a specification must compile against the version it documents:

1. Extract each ` ```ballerina ` block into its own `.bal` file of one scratch package.
2. Wrap fragments so they compile: statements into a `function exN() returns error? { … }`, resource methods into a service with a unique base path, and add stub functions for helpers that the example only calls.
3. Rename clashing module-level names, such as listeners or clients declared in several examples, in the scratch copy only.
4. Give each file only the imports it uses, then `bal build`. Warnings for unused variables are acceptable; errors are not.

Any error is a defect in the example. Fix the example in the specification, not the scratch copy, and rebuild.
