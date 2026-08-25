<!-- markdownlint-disable -->

# Hardening Report: dominikh--staticcheck-action/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dominikh--staticcheck-action/v1.4.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in the composite action step contains two script-injection violations:

(a) Sub-rule (a) — Direct expression interpolation: `${{ runner.temp }}` is interpolated directly inside the shell `run:` script: `export STATICCHECK_CACHE="${{ runner.temp }}/staticcheck"`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because YAML template substitution occurs before the shell ever sees the string, allowing an attacker-controlled or context-derived value to break out of the intended string context.

(b) Sub-rule (b) — Unquoted shell variable expansion: `${merge}` is used unquoted in the command `$(go env GOPATH)/bin/staticcheck -checks "${checks}" -f "${format}" -merge ${merge} | write_output`. The `merge` env var holds `${{ inputs.merge-files }}` (caller-controlled), and the bare unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) embedded in the input to be interpreted by the shell, enabling command injection.

Locations:

- `action.yaml:110`
- `action.yaml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection issues in action.yaml:

(a) Moved `${{ runner.temp }}` out of the `run:` shell script and into the step's `env:` block as `runnerTemp: ${{ runner.temp }}`. The shell script now uses `${runnerTemp}` (a plain env var) instead of the template expression, eliminating direct expression interpolation in the shell.

(b) Replaced the unquoted `${merge}` expansion (which allowed shell metacharacter injection from the caller-controlled `merge-files` input) with a safe bash array built via a `while IFS= read -r` loop that splits the newline-separated file list. The array is then expanded as `"${merge_files[@]}"` (properly quoted), ensuring each file path is treated as a separate, safe argument.

