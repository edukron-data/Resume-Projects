# Day 3: Shell Scripting — Interview Questions and Answers

## Interview questions

### Q1. What is the purpose of a shebang?
**Answer:**

A shebang selects the interpreter when an executable script is launched  
directly. `#!/usr/bin/env bash` locates Bash through PATH, while  
`#!/bin/bash` selects a fixed path. Bash features such as arrays and `[[  
... ]]` should not be assumed to work in a script executed by `sh`.

I match the declared interpreter with the features actually used in the  
script and test it the same way automation invokes it. Running `sh script.sh`  
overrides the shebang choice. For portable scripts, I avoid Bash-specific  
syntax and explicitly document the supported shell and operating environment.

### Q2. How do variables, quoting, and environment variables work?
**Answer:**

Assign with `name=value`, without spaces around `=`. Use `"$name"` to preserve  
whitespace and prevent glob expansion. Single quotes preserve literal text. `export  
NAME=value` makes a variable available to child processes. Quote command  
substitution such as `result="$(command)"` when preserving its output as one value.

For example, `cp "$source" "$destination"` preserves paths containing spaces. I  
distinguish an empty value from an unset variable when either has business meaning.  
Environment variables pass configuration to child processes, but sensitive values still  
require careful handling because diagnostics or subprocess behavior can expose them.

### Q3. How do you use conditions and operators?
**Answer:**

`if [[ -f "$path" ]]; then ...; fi` checks a file in Bash. Use `-eq`,  
`-gt`, and related operators for integer comparisons, or arithmetic  
expressions such as `(( count > 3 ))`. Use string comparisons  
deliberately; a string test and a numeric test answer different questions.

I keep conditions explicit and validate input before arithmetic or filesystem  
operations. For example, a disk threshold must be numeric and within a sensible  
range. Combining tests with logical operators can simplify code, but complex conditions  
are clearer when separated into named checks with useful failure messages.

### Q4. Which loops do you use in Bash?
**Answer:**

Use `for` for a known collection and `while` for repeated input or  
conditions. `while IFS= read -r line; do ...; done < file` preserves  
leading whitespace and backslashes. Avoid `for x in $(cat file)`  
because it splits on whitespace and expands wildcard characters.

When reading files, I decide how blank lines, delimiters, and a final  
line without a newline should be handled. A loop should not silently  
discard meaningful input. I also avoid changing variables inside a  
pipeline subshell when the calling shell needs those values afterward.

### Q5. How do functions, arguments, and arrays work?
**Answer:**

A function receives its own positional arguments: `check() { printf  
'%s\n' "$1"; }`. Script arguments use `$1`, `$2`, and `$#`; `"$@"` preserves  
each argument separately. Bash arrays use `items=(one 'two words')`,  
and `"${items[@]}"` preserves individual elements during iteration.

Functions should perform one clear task and report success or failure through  
their return status. I use local variables inside functions to reduce unintended  
state changes. Arrays are useful for safely constructing command arguments,  
especially when individual arguments contain spaces or optional flags are included.

### Q6. What do exit codes and shell error options mean?
**Answer:**

Exit code zero conventionally means success; nonzero indicates failure or another  
documented condition. `$?` contains the previous command's status. `set -u` detects  
unset variables and `set -o pipefail` propagates pipeline failures. `set -e` has  
contextual exceptions, so critical operations still need explicit error handling.

I capture a command's status immediately because another command replaces `$?`.  
Expected outcomes, such as a search finding no match, need different handling  
from operational failures. Cleanup belongs in deliberate control flow or  
traps so temporary resources are removed even when an important operation fails.

### Q7. How do grep, awk, and sed differ?
**Answer:**

`grep` selects matching lines, `awk` processes fields and records, and  
`sed` performs stream editing. For example, `grep -F 'ERROR' app.log`, `awk  
'{print $1}' file`, and `sed 's/old/new/g' file` solve different tasks.  
Use a JSON parser instead of field splitting for structured JSON logs.

For example, grep finds relevant error lines, awk aggregates a known status  
field, and sed performs a predictable replacement. I confirm the data  
format before selecting a tool. When editing configuration, I write to a  
temporary file and validate the result before replacing the active file.

### Q8. When do you use cut, sort, and uniq?
**Answer:**

`cut -d: -f1 /etc/passwd` extracts a delimiter-separated field. `sort  
-n` sorts numerically and `sort -r` reverses order. `uniq -c` counts  
adjacent identical lines, so unsorted data normally needs `sort` first.  
Simple delimiter tools do not correctly parse every CSV quoting rule.

For example, `sort file | uniq -c | sort -nr` can rank repeated entries. Locale  
can change sorting behavior, so automation should select a consistent  
locale when order matters. If records contain quoted delimiters or embedded  
newlines, I use a format-aware parser instead of treating them as simple fields.

### Q9. How do find and xargs handle unusual filenames?
**Answer:**

`find /logs -type f -name '*.log' -print0 | xargs -0 ...` uses  
NUL delimiters so spaces and newlines in filenames remain intact.  
`find ... -exec command {} +` is another option. GNU `xargs -r`  
avoids running the command on empty input; portability varies.

Before adding a destructive command, I print the selected files and inspect  
a representative sample. NUL delimiters preserve filename boundaries but do  
not make an incorrect selection safe. I also consider permissions, symlinks,  
empty matches, and whether the target tree could change while the script runs.

### Q10. How do you make automation safe to rerun?
**Answer:**

Make operations idempotent: check actual state, create only missing resources,  
and update only changed values. Use temporary files, atomic replacement where  
supported, input validation, and clear exit codes. Log the intended action and  
result, and protect concurrent execution with a lock when shared state is modified.

For example, creating a directory should succeed when it already exists  
in the expected form, but fail if the path is an incompatible file. Idempotency  
is about the resulting state, not suppressing every error. I distinguish  
already-correct state from an operation that could not be completed.

### Q11. How do you debug a shell script?
**Answer:**

Use `bash -n script.sh` for syntax checks and ShellCheck when available. Add  
targeted diagnostics and inspect command exit codes. `bash -x` traces expanded  
commands but can expose credentials, so disable tracing around sensitive operations  
and sanitize logs. Reproduce with the same user, environment, and inputs.

I first reproduce the failure with minimal input and capture the exact  
error before changing several things at once. Syntax checks do not detect  
every runtime problem. After correcting the cause, I test both normal and  
failure inputs and verify that callers receive the intended exit status.

### Q12. How would you count server errors in an access log?
**Answer:**

First confirm the log format. For a conventional combined access  
log with HTTP status in field nine, `awk '$9 ~ /^5[0-9][0-9]$/  
{n++} END {print n+0}' access.log` counts 5xx responses. For JSON  
logs, use the documented status property through a JSON parser.

I validate the field position using actual log samples because proxy formats  
and application logs vary. For operational reporting, I also define the time  
window and whether retries count separately. A correct command against the  
wrong format or time range can produce a convincing but misleading result.

## Scenario-based questions

### S1. Write a script that returns a failure when disk usage crosses a threshold.
**Answer:**

Validate the threshold and query the intended  
filesystem. This Bash example reports a machine-readable  
exit status; a caller can turn it into an alert.  
A query failure must not be reported as healthy.

In an interview, I explain why the script returns distinct statuses: healthy,  
threshold exceeded, and invalid input or measurement failure. The caller  
can then route alerts appropriately. I would test values below, at, and above  
the threshold and simulate a failed filesystem query before scheduling it.

```bash
#!/usr/bin/env bash
set -u -o pipefail
threshold=${1:-85}
[[ $threshold =~ ^[0-9]{1,2}$|^100$ ]] || exit 2
usage=$(LC_ALL=C df -P /var | awk 'NR==2 {gsub(/%/, "", $5); print $5}') || exit 2
[[ $usage =~ ^[0-9]+$ ]] || exit 2
if (( usage >= 10#$threshold )); then
  printf 'ALERT: /var usage is %s%%\n' "$usage"
  exit 1
fi
printf 'OK: /var usage is %s%%\n' "$usage"
```

### S2. A script reports success even though the first command in a pipeline failed. Why?
**Answer:**

By default, a pipeline's status is the last command's status. Enable `set -o pipefail`  
in Bash and check the pipeline's status explicitly. Inspect `PIPESTATUS` immediately if  
individual statuses matter. Account for expected nonzero results, such as `grep` returning  
1 for no matches, so a valid empty result is distinguished from an execution error.

For example, a failed download piped to a successful text filter can otherwise  
appear successful overall. I inspect the important operation directly when  
possible. I also remember that a pipeline may intentionally stop early, so SIGPIPE  
and expected filter behavior need context rather than automatic blanket retries.

### S3. A cleanup script deletes the wrong directory. How would you redesign it?
**Answer:**

Require a fixed allowed root, resolve and validate the candidate path, reject empty paths  
and unexpected symlink targets, and list selected files before enabling removal. Quote  
expansions and use `--` where supported. Apply retention only to the approved subtree and  
file types. Test the selection on disposable fixtures and record the exact deleted paths.

I would include a dry-run mode and require an explicit configured retention rule  
before removal is enabled. The script should log what it selected and why.  
Recovery expectations must be clear because filename validation prevents targeting  
mistakes but cannot restore intentionally deleted data without a usable backup.

### S4. An API automation script occasionally fails. How do you add reliable retries?
**Answer:**

Set connection and overall timeouts, inspect HTTP status and transport errors, and retry only  
transient failures with bounded exponential backoff and jitter. Honor `Retry-After` where supplied. For  
mutating requests, use an idempotency key or verify whether the operation already succeeded before  
retrying. Return a nonzero status after exhaustion and log a correlation ID without credentials.

I keep retry limits and overall execution time visible so automation cannot hang  
indefinitely. Authentication errors and invalid requests usually need correction rather  
than repetition. When the operation succeeds after a retry, logs should still show the  
transient failure and correlation details needed to diagnose recurring instability.
