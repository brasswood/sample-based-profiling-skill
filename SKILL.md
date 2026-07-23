---
name: sample-based-profiling
description: Analyze where a program spends time using Samply, a sampling CPU
profiler. Use to find hot subroutines or to estimate the proportion of time
spent in a given subroutine.
license: Apache 2.0
metadata:
    author: Andrew Riachi
---

# Sample-Based Profiling

Use Samply to measure where a program spends its time.

## Important note on Windows

**If you are on Windows, you must run Samply outside of the sandbox.** Samply
requires elevation because it uses ETW. If you run it inside the sandbox, the
UAC prompt will fail, then Samply will hang indefinitely instead of exiting
with an error. If you are curious for more information/updates about this
issue, see https://github.com/mstange/samply/issues/842.

## Get a profile

```powershell
samply record --save-only -o <OUT_PROFILE.json.gz> -- <COMMAND_TO_PROFILE> <ARGS>
```

Prefer a self-contained symbolicated profile when the installed Samply supports
`--presymbolicate`:

```powershell
samply record --presymbolicate --save-only -o <OUT_PROFILE.json.gz> -- <COMMAND_TO_PROFILE> <ARGS>
```

Check `samply record --help` rather than assuming the flag is available. Older
Samply versions may instead offer `--unstable-presymbolicate`, which writes a
`.syms.json` sidecar rather than embedding symbols; preserve the sidecar with
the profile.

In Rust projects, if `cargo samply` is available:

```powershell
cargo samply --samply-args="[--presymbolicate] --save-only -o <OUT_PROFILE.json.gz>"
```

`--save-only` prevents the browser UI from opening automatically after the
profile completes. You should use `profiler-cli` instead, described next.

## Read a profile

Use the Firefox Profiler CLI (`@firefox-devtools/profiler-cli` on npm) to read
the generated profile JSON. It is a terminal interface for profile inspection
and can emit JSON suitable for further processing:

```powershell
profiler-cli load profile.json
profiler-cli thread info
profiler-cli thread samples
profiler-cli thread samples-top-down
profiler-cli thread samples-bottom-up
profiler-cli thread functions --json
```

Discover the installed version and its help before relying on a command or
output shape.

If `profiler-cli` is not installed, run it through `npx`, install it globally,
or fall back to another method for reading profiles.

## Symbolicate native frames

Treat executable offsets such as `program.exe!0x1234`, or searches that cannot
find expected native functions, as evidence that symbolication is incomplete.

If the profile was not presymbolicated, run Samply's local symbol service
without opening the browser:

```powershell
samply load --no-open --symbol-dir <DIRECTORY_WITH_BINARY_AND_DEBUG_FILES> --port <PORT> profile.json.gz
```

(`--no-open` prevents opening the profiler UI while the local process serves
the profile, symbols, and source.)

Keep this process alive. It prints a profiler URL containing a percent-encoded
`symbolServer` value. Decode that value and pass it to the CLI:

```powershell
profiler-cli load profile.json.gz --symbol-server <DECODED_SYMBOL_SERVER_URL>
```

Ensure the binary and debug files match the recorded build. For Windows Rust
builds, point `--symbol-dir` at the directory containing the matching `.exe`
and `.pdb`. If the environment terminates background processes between shell
calls, start Samply and run the CLI queries within the same long-lived shell.

## Preserve latitude

- Use the CLI when its hot-function table, call tree, filters, or JSON answer
  the question.
- Inspect the profile format or seek another compatible analysis tool when the
  CLI is unavailable or insufficient.
- Record which artifact, commands, filters, and assumptions led to a
  conclusion, so another agent can reproduce or challenge it.
