<!-- markdownlint-disable -->

# Hardening Report: dominikh--staticcheck-action/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dominikh--staticcheck-action/v1.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ runner.temp }}` is directly interpolated inside a `run:` shell command string at line 106: `export STATICCHECK_CACHE="${{ runner.temp }}/staticcheck"`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell script before the shell parses it. The safe pattern is to use the corresponding environment variable (`$RUNNER_TEMP`) instead of the expression form.

Locations:

- `action.yaml:106`

### script-injection (severity: high)

Sub-rule (b): The shell variable `${merge}` — which holds the value of `inputs.merge-files` (untrusted caller-supplied input) — is expanded **unquoted** inside the `run:` block at line 120: `$(go env GOPATH)/bin/staticcheck -checks "${checks}" -f "${format}" -merge ${merge} | write_output`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) out of the value, enabling command injection. It must be double-quoted: `-merge "${merge}"`.

Locations:

- `action.yaml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection issues in action.yaml:
1. Line 106: Replaced `${{ runner.temp }}` with `$RUNNER_TEMP` (the pre-set environment variable) to eliminate GitHub Actions expression interpolation inside the run block.
2. Line 120: Replaced the unquoted `${merge}` expansion (a newline-separated list of files) with a safe bash array. The code now reads each newline-delimited file path into a `merge_files` array via `while IFS= read -r merge_file`, then passes `"${merge_files[@]}"` to staticcheck — preserving correct multi-file argument splitting while preventing shell metacharacter injection.

