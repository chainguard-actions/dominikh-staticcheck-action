<!-- markdownlint-disable -->

# Hardening Report: dominikh--staticcheck-action/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dominikh--staticcheck-action/v1.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ runner.temp }} expression is directly interpolated inside the run: shell script: `export STATICCHECK_CACHE="${{ runner.temp }}/staticcheck"`. Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yaml:110`

### script-injection (severity: high)

Sub-rule (b): The shell variable ${merge} — sourced from inputs.merge-files (attacker-controlled) via the env: block — is used unquoted in the command: `$(go env GOPATH)/bin/staticcheck -checks "${checks}" -f "${format}" -merge ${merge} | write_output`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) from the value, enabling command injection. It should be `"${merge}"`.

Locations:

- `action.yaml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yaml:
1. Moved `${{ runner.temp }}` from the inline run: shell script to the env: block as `RUNNER_TEMP: ${{ runner.temp }}`, then referenced it as `${RUNNER_TEMP}` in the shell script.
2. Replaced the unquoted `${merge}` expansion (which allowed shell metacharacter injection from the attacker-controlled inputs.merge-files) with a bash array: the newline-separated list is read line-by-line into `merge_files=()` using `while IFS= read -r merge_file`, then passed to staticcheck as `"${merge_files[@]}"` so each file path is a separate, properly quoted argument.

