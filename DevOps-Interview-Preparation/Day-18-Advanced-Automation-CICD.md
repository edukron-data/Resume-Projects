# Day 18: Advanced Automation and CI/CD — Interview Questions and Answers

## Interview questions

### Q1. How do you choose Bash, PowerShell, or Python?
**Answer:** Bash suits Linux command composition, PowerShell offers structured objects and strong Windows/Azure administration support, and Python suits larger logic, APIs, data handling, and reusable libraries. Choose according to complexity and maintenance skills rather than forcing every task into the team's first scripting language.

### Q2. What makes a production automation script reliable?
**Answer:** Validate inputs, use explicit identity and environment selection, define timeouts, handle expected failures, and return meaningful exit codes. Support safe reruns, bounded retries, structured diagnostics, and cleanup. Document what state can be changed and how to recover after partial completion.

### Q3. How should PowerShell handle errors?
**Answer:** Use `try`/`catch` for terminating errors and `-ErrorAction Stop` where nonterminating cmdlet errors must be caught. Inspect native-command exit codes through `$LASTEXITCODE` because behavior differs from cmdlet errors and between versions/settings. Avoid catch blocks that log a message and accidentally return overall success.

### Q4. What should Python automation include?
**Answer:** Use an argument parser, explicit configuration, timeouts for external calls, appropriate exception handling, and nonzero process exit statuses on failure. Manage dependencies reproducibly and avoid shell invocation when a library or argument-list subprocess call is suitable. Redact credentials from logs and test important failure paths.

### Q5. How do you safely automate Azure CLI operations?
**Answer:** Authenticate through the intended workload identity, verify the subscription, use structured JSON output, and parse known properties. Check exit statuses and wait for required asynchronous operations. Query current state before mutation and handle retries carefully so a timeout after creation does not produce duplicate resources.

### Q6. How should reusable YAML templates be versioned?
**Answer:** Give shared templates a documented input contract and pin consumers to an intentional version/ref where supported. Validate proposed changes against representative consumers before rollout. Publish breaking changes deliberately and provide a migration path so a central edit cannot unexpectedly break every release pipeline.

### Q7. How do you secure self-hosted agents?
**Answer:** Separate trust levels and environments, use least-privilege agent identities, patch tools/OS, restrict network access, and clean workspaces. Prefer disposable agents where practical. Do not run untrusted PR code on an agent with production access; job separation alone does not erase machine-level persistence.

### Q8. How do you optimize pipelines without weakening quality?
**Answer:** Measure the critical path, reduce unnecessary downloads and artifact sizes, cache dependencies correctly, and parallelize independent work within capacity. Separate queue delays from execution time. Preserve mandatory security and correctness gates, and ensure path-based optimization does not skip checks needed by shared-library changes.

### Q9. How do you implement a blue-green deployment?
**Answer:** Provision compatible green capacity, deploy the new artifact, warm it, validate it, and switch traffic using the selected platform's routing mechanism. Observe health and keep blue available for a defined recovery window. Manage background jobs, shared state, sessions, and database compatibility before assuming traffic reversal is safe.

### Q10. How would you automate canary analysis?
**Answer:** Define a controlled traffic cohort, baseline, observation window, minimum sample size, and failure thresholds. Compare error rate, latency, and relevant business outcomes. Automate pause/rollback when criteria are met and keep a human override under the incident process. Traffic weighting needs a capable router or rollout controller.

### Q11. What is idempotency in infrastructure automation?
**Answer:** Repeating an operation with the same desired inputs leads to the same intended state without duplicate side effects. Use stable resource identities, state checks, conditional updates, and supported idempotency keys. A retry loop is safe only if duplicate or partially completed operations are accounted for.

### Q12. How do you handle automation concurrency?
**Answer:** Identify shared mutable state and apply the appropriate lock, lease, transaction, or compare-and-swap mechanism. Assign unique temporary paths and isolate independent jobs. Include lock expiry/recovery design where supported, but never assume an expired client proves the server-side operation has stopped.

## Scenario-based questions

### S1. A self-hosted agent succeeds only on every second run. How do you diagnose it?
**Answer:** Suspect residual files, processes, ports, caches, or credentials. Compare a failed run with a successful one and reproduce on a clean agent. Inspect workspace assumptions and cleanup behavior, then make all required inputs explicit and outputs deterministic. If possible, use disposable agent instances. Validate several clean and repeated runs after the fix, including failure recovery.

### S2. A script times out after creating a resource, then retries and creates duplicates. How do you fix it?
**Answer:** Use a deterministic resource identifier or API idempotency key and distinguish a client timeout from confirmed server failure. After an ambiguous result, query operation/resource status before creating again. Record correlation IDs and apply bounded retries. Reconcile existing duplicates through an ownership-aware cleanup plan, and test the ambiguous-timeout path so the behavior is safe under network interruption.

### S3. A shared template update breaks ten pipelines. What is your response?
**Answer:** Stop wider adoption, identify affected consumers, and restore or pin the last compatible template revision where safe. Compare the changed contract, default values, task versions, and expansion behavior. Validate a corrected version with representative consumers, then roll it out in stages. Add compatibility checks and versioning rules so a central update cannot silently alter every pipeline at once.

### S4. Blue-green traffic switches successfully, but users lose sessions. Why?
**Answer:** Session state may be stored only in old instances, or cookie encryption/signing keys may differ between environments. Verify session-store configuration, key continuity, cookie/domain settings, and connection draining. Externalize or deliberately migrate session state where appropriate and test authenticated user journeys during a switch. A successful health probe does not demonstrate session continuity or background-job correctness.
