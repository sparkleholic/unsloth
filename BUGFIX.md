# llama-server macOS launch bug

## Summary

Unsloth Studio failed to start `llama-server` on macOS even though the same command worked when run directly in a terminal.

The root cause was not the GGUF file, `mmproj`, `flash-attn`, or the model arguments.
The root cause was the Studio launch path itself.

## Symptom

Studio logs showed immediate process death:

```text
llama-server exited with code -11
```

At the same time:

- running the exact `llama-server ...` command in the shell worked
- a minimal Python `fork/exec` repro also worked
- `llama-server --version` could be made to work once the launch path was simplified

This isolated the issue to how Studio started the child process on macOS.

## Root cause

The original backend used `subprocess.Popen(...)` for the actual `llama-server` launch.

In this macOS environment, that Studio-side launch path was unstable, while a minimal direct process launch was stable.
During debugging, several wrapper-based experiments were tried:

- helper Python process
- `env -i`
- shell launch
- PTY/script launch
- option retries without `mmproj`
- option retries without `flash-attn`

Those were diagnostic only. They were not the real fix.

The useful result was much simpler:

- direct shell launch worked
- minimal Python `fork/exec` launch worked

Therefore the necessary fix was to make Studio use a similarly minimal native launch path on macOS.

## Final fix kept

Only the following changes were kept:

1. `studio/backend/core/inference/llama_cpp.py`

   - On macOS, launch `llama-server` via direct `os.posix_spawn(...)`.
   - Redirect stdout/stderr to a temp log file for startup diagnostics.
   - Read back the temp log tail if startup fails before health check succeeds.

2. `studio/backend/core/inference/llama_cpp.py`

   - Fix `-c` handling so the requested `n_ctx` is respected.
   - Previously the code always passed `-c 0`, which means "use model native context".
   - For this model that meant a 262144-token context, which was a separate real bug.

## Changes explicitly reverted

The following debugging changes were removed because they were not required for the final fix:

- helper launcher file
- `--version` probe path
- `env -i` helper execution
- shell/PTY/script launch experiments
- option retry matrix
- route-level API response tweaks related to temporary fallback behavior
- temporary repro script under `_app`

## Why this is the minimal correct fix

The final code now changes only the part that was actually broken:

- macOS child-process launch path

It does not change model options, feature flags, or API semantics unless necessary.

## Result

Studio now uses a macOS launch path that matches the behavior of the known-good minimal Python repro closely enough to avoid the crash, while keeping the rest of the inference flow unchanged.
