<!-- markdownlint-disable -->

# Hardening Report: dominikh--staticcheck-action/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dominikh--staticcheck-action/v1.4.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A `${{ runner.temp }}` expression is directly interpolated inside a `run:` shell script: `export STATICCHECK_CACHE="${{ runner.temp }}/staticcheck"`. Any `${{ ... }}` expression embedded directly in a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. Sub-rule (b): The env var `${merge}` (sourced from `inputs.merge-files`) is used **unquoted** in the shell command `$(go env GOPATH)/bin/staticcheck -checks "${checks}" -f "${format}" -merge ${merge} | write_output`. An unquoted expansion allows an attacker-controlled value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) to be interpreted by the shell, enabling command injection.

Locations:

- `action.yaml:109`
- `action.yaml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection issues in action.yaml:
1. Moved `${{ runner.temp }}` out of the `run:` shell block and into the `env:` block as `STATICCHECK_CACHE: ${{ runner.temp }}/staticcheck`. Removed the `export STATICCHECK_CACHE=...` line from the shell script since the env block sets it directly.
2. Quoted the unquoted `${merge}` variable in the `-merge ${merge}` shell command, changing it to `-merge "${merge}"` to prevent shell metacharacter injection from attacker-controlled `inputs.merge-files` values.

