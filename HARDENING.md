<!-- markdownlint-disable -->

# Hardening Report: dominikh--staticcheck-action/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dominikh--staticcheck-action/v1.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The expression `${{ runner.temp }}` is directly interpolated inside a `run:` shell command string: `export STATICCHECK_CACHE="${{ runner.temp }}/staticcheck"`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yaml:111`

### script-injection (severity: high)

Rule (b): The shell variable `${merge}` — which holds the user-controlled input `${{ inputs.merge-files }}` — is expanded **unquoted** in the command: `$(go env GOPATH)/bin/staticcheck -checks "${checks}" -f "${format}" -merge ${merge} | write_output`. An unquoted expansion allows the shell to parse metacharacters (spaces, globs, semicolons, `$(...)`, etc.) out of the value, enabling command injection via the `merge-files` input.

Locations:

- `action.yaml:122`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yaml:
1. Moved `${{ runner.temp }}` out of the `run:` block into the step's `env:` block as `RUNNER_TEMP`, then referenced it as `${RUNNER_TEMP}` in the shell script.
2. Fixed unquoted `${merge}` expansion for the `merge-files` list input: replaced `-merge ${merge}` with a bash array built via `while IFS= read -r merge_file` loop over the newline-separated input, then expanded as `-merge "${merge_files[@]}"` to keep each file path as a separate, properly quoted argument.

