# Day 7: Azure CI Pipelines — Interview Questions and Answers

## Interview questions

### Q1. Explain stages, jobs, steps, and tasks.
**Answer:**

A stage groups a major phase such as Build or Test. A job groups steps  
executed by an agent or supported server context. A step is a script  
or task. Tasks are reusable packaged operations. Jobs can run on  
different machines, so local files do not automatically carry across them.

For example, separate build and deployment jobs may use different agents and  
permissions even when they belong to one pipeline. Each job should explicitly obtain  
its required inputs. Understanding the hierarchy also helps place variables,  
conditions, timeouts, and approvals at the scope where they have the intended effect.

### Q2. How do YAML triggers work?
**Answer:**

CI triggers select branches and optionally paths for push-triggered runs. Schedules, pipeline-resource triggers,  
and manual runs serve other workflows. For Azure Repos Git, PR validation is configured through branch  
build-validation policy rather than YAML `pr` triggers. Make trigger intent explicit and test exclusions. See [PR  
trigger support](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/pr?view=azure-pipelines).

I inspect the run reason when a pipeline starts unexpectedly because the same  
YAML can be activated by several mechanisms. Branch and path filters should  
reflect actual dependency relationships. A change to shared tooling may need  
broader validation than a change to one isolated application's documentation.

### Q3. How do variables differ from parameters?
**Answer:**

Variables are string-based values usable during pipeline execution. Runtime  
parameters are typed inputs expanded while the pipeline plan is constructed. Use  
parameters to select template structure and variables for execution values.  
Parameters are not a secret-storage mechanism and should not receive credentials.

For example, a boolean parameter can decide whether a template includes a testing  
job, while a variable can hold a run identifier. I consider when the value  
becomes available before choosing the mechanism. Secret values belong in  
protected storage, not in a parameter exposed through run inputs or expanded YAML.

### Q4. Compare template expressions, macro syntax, and runtime expressions.
**Answer:**

`${{ ... }}` is evaluated during template expansion and can use parameters. `$(name)` macro syntax substitutes  
variables before tasks execute. `$[ ... ]` evaluates runtime expressions, commonly for conditions or variable  
assignment. A value produced by a task cannot influence a template decision already expanded earlier. See [Azure  
expression timing](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops).

A common mistake is expecting a variable produced during execution to change an  
earlier template expansion decision. I separate pipeline construction from job  
execution in my explanation. For debugging, I inspect the value at its actual  
evaluation scope and avoid printing sensitive variables to understand timing.

### Q5. Why use pipeline templates?
**Answer:**

Templates reuse reviewed steps, jobs, or stages and reduce inconsistent security  
and build practices. Define typed inputs, sensible defaults, and a stable versioning  
strategy. Excessive abstraction can hide execution behavior, so keep templates  
understandable and document required repositories, permissions, and tool versions.

For example, a standard build template can require test publication and approved scanning  
while accepting the application path and runtime version as inputs. Consumers should  
understand which behavior they can override. I keep security-critical decisions under  
an appropriate owner rather than making every protective step an optional parameter.

### Q6. How do dependencies and conditions work?
**Answer:**

`dependsOn` expresses ordering between jobs or stages. Conditions decide whether  
execution proceeds. Default success behavior can be replaced by a custom condition,  
so include `succeeded()` when successful dependencies remain required. Use  
deliberate handling for failure notifications, canceled runs, and always-run cleanup.

For a deployment stage, I usually require both successful prerequisites and an  
explicit branch or environment condition. A notification job may intentionally run  
after failure. I test canceled and skipped cases too because a condition that works  
on successful runs can behave unexpectedly when an upstream stage never executes.

### Q7. How do you pass files between jobs?
**Answer:**

Publish the required files as an artifact and download them in the consuming  
job. Identify artifacts by the intended pipeline run, not a loosely selected  
latest output. Self-hosted agents may retain files, but relying on residue  
creates nondeterministic builds and can accidentally deploy stale content.

I publish only the files required by downstream consumers and document the  
artifact's layout. This reduces transfer time and accidental secret exposure.  
The deployment should fail clearly when its expected artifact is missing  
rather than fall back to a leftover local file or a different successful run.

### Q8. How do you pass an output variable to another job?
**Answer:**

Have a named step set a variable with the `isOutput=true` logging-command flag.  
Map it in the dependent job using the appropriate `dependencies` expression;  
cross-stage access uses `stageDependencies` in supported contexts. Syntax  
depends on the producer and consumer scopes, so verify it with a small pipeline.

The producer step needs a stable name, and the consuming job must depend  
on the correct producer. I verify the variable's scope and expression syntax  
with a nonsecret example. If the value represents release identity, I also  
validate it before using it to select an image, package, or environment.

### Q9. What belongs in a dependency cache?
**Answer:**

Cache reusable dependency downloads to improve speed, keyed by operating system,  
dependency manager, and lockfile content. A cache is an optimization; the  
build must still work after a miss. Publish release outputs as artifacts rather  
than treating caches as authoritative or security-validated deployment content.

For example, changing a dependency lockfile should select a different dependency  
cache instead of silently reusing stale downloads. A restored cache must not  
bypass integrity checks. I measure cache transfer cost as well because a very  
large cache can make the pipeline slower than a clean dependency installation.

### Q10. What should a useful CI pipeline do?
**Answer:**

Check out source, install pinned tools and dependencies, run linting and meaningful  
tests, scan relevant risks, build once, and publish a uniquely identifiable  
artifact. Publish test results and preserve useful diagnostics. Fail the run when a  
required quality check fails instead of hiding failures with unconditional success.

The artifact should carry enough metadata to connect it to source, dependencies,  
tests, and the build run. This supports release investigation and rollback.  
Required checks need actionable failures with retained diagnostics; otherwise  
teams may learn to ignore unreliable red builds and weaken the delivery process.

### Q11. How do you handle secrets in CI?
**Answer:**

Use protected secret variables, a secret store, or federated identity as  
appropriate. Map secrets only into steps that need them and avoid echoing  
command lines or enabling tracing. Masking is a secondary safeguard, not  
a guarantee. Untrusted PR code should not receive production credentials.

I ensure forks or untrusted branches cannot access protected values through  
scripts, dependency hooks, or modified tests. Even a command that never echoes a  
secret can exfiltrate it if malicious code runs with access. Separate trusted  
release execution from untrusted validation and audit changes to those boundaries.

### Q12. What affects reproducible builds?
**Answer:**

Pin dependency versions with lockfiles, control toolchains and base images, record  
source revisions, and avoid undeclared dependencies on agent state. External  
registries and timestamps can still affect outputs. Record provenance and artifact  
digests so the deployed build can be traced and reproduced as closely as required.

I also remove implicit downloads of whatever version happens to be current and record  
the actual tool versions used. Clean execution environments reveal missing inputs that  
persistent agents can hide. For strict reproducibility, I assess nondeterministic timestamps  
and build tooling rather than assuming a lockfile alone guarantees identical bytes.

## Scenario-based questions

### S1. A pipeline runs twice for one change. How do you diagnose it?
**Answer:**

Inspect each run's reason and triggering resource. Check push triggers, Azure Repos PR build  
policies, schedules, upstream pipeline triggers, and duplicate pipeline definitions. A feature-branch  
push and PR validation may legitimately both run. Decide which checks are required, adjust the  
overlapping trigger or policy, and test a new push and PR update to confirm the intended behavior.

I compare whether both runs are needed: one may validate branch changes while the  
other validates a PR integration. The goal is intentional coverage, not simply fewer  
runs. After adjusting settings, I check that protected branches still receive  
mandatory validation and that routine changes no longer cause unwanted duplicate work.

### S2. Tests pass locally but fail on the hosted agent. What do you check?
**Answer:**

Compare OS, case sensitivity, runtime versions, locale, timezone, environment variables,  
dependencies, and working directory. Reproduce using a clean install from the lockfile. Inspect  
published test results and logs, especially assumptions about local services and files. Fix  
the hidden dependency or environment mismatch; do not simply rerun until a flaky test passes.

Case-sensitive paths and hidden local dependencies are common causes when moving from  
a workstation to Linux agents. I reproduce one failing test with matching versions  
and inspect its assumptions. If the test is genuinely timing-sensitive, I fix  
synchronization or isolation rather than adding arbitrary delays that conceal the issue.

### S3. A deployment job cannot find a build artifact. What is likely wrong?
**Answer:**

Verify the build published the expected artifact name and path, and that publishing was  
not skipped after a failed step. Confirm the deployment selected the intended run and  
downloaded into the path it uses. Check retention and permissions. List the downloaded files  
in a safe diagnostic step and fail explicitly if the required release file is absent.

A build job and deployment job may use different working directories, so I  
compare their expanded paths explicitly. I also check whether the artifact came  
from the expected branch and revision. Correcting a path is insufficient if  
the pipeline still selects the wrong release content under concurrent runs.

### S4. Build time grows from 8 minutes to 25 minutes. How would you optimize it?
**Answer:**

Compare task durations before and after the change. Distinguish queue time, dependency downloads,  
compilation, tests, and artifact uploads. Fix incorrect cache keys or oversized build contexts, parallelize  
independent tests within capacity, and avoid duplicate work. Keep required checks intact, then measure several  
representative runs to confirm improvement and check that cache misses still produce correct builds.

I choose improvements from timing evidence rather than adding caching or more  
agents by default. For example, parallel test shards help only if setup overhead  
and capacity permit a shorter critical path. I verify correctness, representative  
duration, and cost after the change, including a run where caches are cold.
