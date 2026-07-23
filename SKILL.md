---
name: sample-based-profiling
description: Analyze where a program spends time using Samply, a sampling CPU profiler. Use to find hot subroutines or to estimate the proportion of time spent in a given subroutine.
compatibility: Works on Windows, Linux, and MacOS. Requires `samply` to be installed somewhere.
license: Apache 2.0
metadata:
    author: Andrew Riachi
---

# Sample-Based Profiling

Use Samply to measure where a program spends its time.

## Important note on Windows

**If you are on Windows, you must run Samply outside of the sandbox.** Samply requires elevation because it uses ETW. If you run it inside the sandbox, the UAC prompt will fail, then Samply will hang indefinitely instead of exiting with an error. If you are curious for more information/updates about this issue, see https://github.com/mstange/samply/issues/842.

## Get a profile

```powershell
samply record --save-only -o <OUT_PROFILE.json.gz> -- <COMMAND_TO_PROFILE> <ARGS>
```

In Rust projects, if `cargo samply` is available:

```powershell
cargo samply --samply-args="--save-only -o <OUT_PROFILE.json.gz>"
```

`--save-only` prevents the browser UI from opening automatically after the profile completes. You should use `profiler-cli` instead, described next.

## Read a profile

Use the Firefox Profiler CLI (`@firefox-devtools/profiler-cli` on npm) to read the generated profile JSON. It is a terminal interface for profile inspection and can emit JSON suitable for further processing:

```powershell
profiler-cli load profile.json
profiler-cli thread info
profiler-cli thread samples
profiler-cli thread samples-top-down
profiler-cli thread samples-bottom-up
profiler-cli thread functions --json
```

Discover the installed version and its help before relying on a command or output shape.

If `profiler-cli` is not installed, ask for permission to install it before falling back to another method for reading profiles.

## Preserve latitude

- Use the CLI when its hot-function table, call tree, filters, or JSON answer the question.
- Inspect the profile format or seek another compatible analysis tool when the CLI is unavailable or insufficient.
- Record which artifact, commands, filters, and assumptions led to a conclusion, so another agent can reproduce or challenge it.

