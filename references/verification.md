# Verification

How `bal-lib-*` skills turn a claim about a library into **verified** evidence (see `findings.md`): run a small program against the library and observe the result. A scratch package is the unit of verification. It takes seconds to build, where a repository test suite can take tens of minutes.

## Which version to run

Run against the released version that matches the code you read:

1. Read the version in `ballerina/Ballerina.toml` and the latest release tag (`git tag --sort=-v:refname | head`).
2. If the working tree is the released code, or differs only in documentation or CI, depend on that version from Ballerina Central.
3. If the code you read contains unreleased behaviour changes, say so in the evidence, and either run against the release (stating the difference) or build the library locally and point the scratch package at the local build. The repository's own build instructions say how; `-PdeleteDependencies` or an equivalent flag is often needed, otherwise the build silently resolves the released version from Central.

## Scratch package

Create each scratch package in its own new directory outside the repository, in the session's scratchpad when one exists:

```
<scratch>/<topic>/
  Ballerina.toml     [package] org = "t", name = "<topic>", version = "0.1.0"
  main.bal
```

- Unused imports are compile errors, so import only what the file uses.
- `bal build` runs the library's compiler plugin, so a successful build also checks compile-time diagnostics. To verify a diagnostic, build a package that should trigger it and read the reported code and location.
- Run the built jar with `java -jar target/bin/<name>.jar`, which avoids rebuilding.

## What to probe

Reading code is least reliable for the following. Probe them first, whatever the library:

- **Boundary and invalid input**: empty, absent, maximum, out-of-range, malformed and multi-byte values, for every parameter and configuration field.
- **Returned error versus panic**: whether each failure is returned to the caller or ends the program. Call without `trap` and check the exit code and stderr; a trapped panic is indistinguishable from a returned error in an `is` check.
- **Shared state**: run at least two concurrent connections, requests, clients or calls, and check that the state of one does not leak into another.
- **Lifecycles**: start, stop, restart, attach and detach sequences, what happens to open connections and pending calls at each step, and calling a method twice or on an object that was never started.
- **Defaults through conversions**: how a configured value is converted on its way to the platform (units, rounding, clamping, `0` and `-1`).
- **Delegated platform behaviour**: when a function wraps a JDK or runtime API, that API's behaviour is the implementation. Run it rather than recalling it.
- **Round trips**: for each pair of inverse operations, whether the result reproduces the input.
- For networking libraries also: dispatching precedence, data binding, status codes and error mapping, wire formats, and header spelling.

## Programs with listeners

For libraries that open listeners:

- Prefer one program: in `main`, attach the service and start the listener explicitly (`check ep.attach(svc); check ep.'start();`), then make the client calls, and stop the listener at the end. Module-level listeners only start after `main` returns, so a module-level service cannot be called from `main` in the same program.
- When the library has no programmatic start, run the service jar in the background and drive it from a second process: `curl` for HTTP, `nc` or a short Python script for raw sockets, or the same jar in client mode guarded by a configurable (`configurable boolean clientMode = false;` with `-CclientMode=true`). The client-mode process also tries to bind the listener after `main` returns, fails with "Address already in use", and may not exit; stop it with `pkill -9 -f <name>.jar`.
- Declare one listener per port. Give each scratch package its own port range, so parallel runs do not collide. To re-run an earlier run's reproducer under a different port range, copy it and change the ports in the copy.
- Wait for readiness by polling rather than sleeping a fixed time. A readiness probe is itself a connection: it triggers connection callbacks and counts in connection-level state, so account for it, or probe after the counters are read.
- Observe the exact output. For HTTP, `curl -s -i`, dropping volatile headers such as `date` and `server`; for raw bytes and line endings, `od -c`.
- Stop every background process and container when done.

## Recording the result

Copy the exact input and the relevant part of the output into the finding or the trace. A claim is verified only when the observed output matches what the claim states. When it does not, the claim is wrong, not the run: fix the claim, and re-read the code to understand why.

A bug's reproducer asserts the specified behaviour and reports PASS or FAIL per check, so one package can reproduce several bugs. For a hang, assert completion within a bound, and stop the process when the bound passes. For a compile-time bug, the reproducer is a package whose build reports the wrong diagnostic, or fails to report the right one.

When the repository has a fast, targeted test runner, a bug's reproducer can also be proposed as a test in the finding, so that it can become the regression test of the fix. Proposing it is part of the finding; adding it to the repository is the developer's decision.

## When a claim cannot be run

Some behaviour needs infrastructure the session lacks: a remote service, several network interfaces, a failing DNS, a stalled peer, a specific operating system, or a timing window that is impractical to hit. Record the claim as **read**, say why it was not run, and list it in the report's coverage. When the behaviour depends on a runtime conversion, such as `decimal` to `int`, inspect the runtime method instead of assuming it, e.g. `javap -c` on the class in the distribution's `bre/lib` jars.

## Compiling and running examples

Every Ballerina example in a specification must compile against the version it documents. Apply the same check to the examples in `README.md` and the API docs, which is cheap and catches documentation bugs.

1. Extract each ` ```ballerina ` block into its own `.bal` file of one scratch package.
2. Wrap fragments so they compile: statements into a `function exN() returns error? { … }`, resource or remote methods into a service or class, and add stub functions for helpers that the example only calls.
3. Rename clashing module-level names, such as listeners or clients declared in several examples, in the scratch copy only.
4. Give each file only the imports it uses, then `bal build`. Warnings for unused variables are acceptable; errors are not.

Any error is a defect in the example. Fix the example, not the scratch copy, and rebuild.

To check the outcomes an example states, write the assertions in a separate file of the scratch package, calling the example code and comparing with the stated results, so the example itself stays minimal. An example that declares a module-level service and calls it from `main` compiles but cannot run; split it into a service example and a client example. An example whose run disagrees with its stated outcome only because of a recorded bug is acceptable when the specification states the intended behaviour; cite the finding in the trace.
