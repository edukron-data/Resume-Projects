# Day 19: Real-Time Production Troubleshooting — Interview Questions and Answers

## Interview questions

### Q1. What is your first response to a production outage?
**Answer:** Confirm impact and scope, establish incident ownership, check recent changes, and preserve useful evidence. Stabilize the service using the lowest-risk available mitigation, such as a safe rollback or traffic shift. Communicate verified facts and next checkpoints while deeper diagnosis continues.

### Q2. How do you distinguish a symptom from a root cause?
**Answer:** A symptom is observed behavior, such as a 503. A cause explains the mechanism, such as no ready backend after a configuration error. Build a timeline and test hypotheses against evidence. Fixing one visible symptom is not proof the underlying failure or recurrence path has been removed.

### Q3. What does CrashLoopBackOff mean?
**Answer:** Kubernetes is backing off repeated container restarts. Inspect current and previous logs, termination reasons, exit codes, events, configuration, memory limits, and probes. Common causes include startup exceptions, missing secrets, OOM kills, or inappropriate liveness checks. The backoff is a consequence, not the root cause.

### Q4. What causes ImagePullBackOff?
**Answer:** The container image cannot be pulled and Kubernetes is retrying with backoff. Events may reveal a missing image/tag, authorization failure, registry DNS/connectivity issue, throttling, or unsupported image platform. Verify the exact image reference and node/runtime identity before changing credentials or redeploying.

### Q5. How do you investigate DNS failure in Kubernetes?
**Answer:** Test lookup from an affected Pod or approved diagnostic container, inspect its resolver configuration, and distinguish cluster-service names from external/private names. Check CoreDNS health, Service/EndpointSlices, network policies, upstream resolvers, and private DNS forwarding. A successful lookup on your laptop does not prove Pod DNS works.

### Q6. What can cause HTTP 503?
**Answer:** The responding component may have no healthy backends, be overloaded, enforce maintenance, or reject traffic due to upstream limits. Identify which component produced the status using headers, logs, and request correlation. Inspect readiness, endpoint availability, capacity, and deployment changes before restarting unrelated services.

### Q7. How do you diagnose a failed pipeline quickly?
**Answer:** Identify the first failing task and its precise error, separating queue/agent issues from script, authentication, build, test, or deployment failures. Compare against the last successful run's source, templates, identity, and tool versions. Retrying is useful only when a transient condition is plausible and the operation is safe to repeat.

### Q8. What do you check when a Linux server becomes unreachable?
**Answer:** Separate DNS, routing, firewall, SSH listener, OS health, and cloud-resource state. Use cloud console diagnostics where authorized to check boot, disk, memory, and service logs. Avoid assuming an SSH timeout means the VM is stopped. Preserve evidence before disruptive recovery actions where feasible.

### Q9. What are common Terraform incident classes?
**Answer:** Wrong backend/workspace selection, stale or active locks, state loss, drift, conflicting ownership, provider/API errors, and partial applies are common classes. Confirm the selected context before mutation, secure a recoverable state version, and review a new plan. Deleting state or disabling locks is not a general repair method.

### Q10. How do you investigate an OOMKilled container?
**Answer:** Inspect termination details, limits/requests, recent memory trends, workload changes, and whether pressure came from the container or node. Analyze heap growth, caches, concurrency, and request size. Increase capacity only when justified, and verify whether the actual problem is a leak or an unsuitable memory budget.

### Q11. When would you roll back instead of debug in place?
**Answer:** Roll back when impact is significant, a recent change is strongly implicated, and a known-good version can be restored safely. Consider schema, configuration, and external effects first. Preserve diagnostics and continue cause analysis after recovery. If rollback is incompatible, use a safer mitigation or forward fix.

### Q12. What belongs in a post-incident review?
**Answer:** Document user impact, timeline, detection, contributing conditions, mitigation, recovery verification, and what helped or hindered response. Assign practical preventive and detection improvements with owners and dates. Focus on system conditions and decisions using available information rather than attributing the incident to one person's mistake.

## Scenario-based questions

### S1. After an AKS deployment, all users receive 503 errors. Walk through your investigation.
**Answer:** Pause rollout and identify whether the 503 comes from the ingress, gateway, or application. Check Deployment availability, Pod readiness, Service selectors/ports, and EndpointSlices. Inspect new Pod logs, events, and changed configuration. If the release removed healthy endpoints, safely restore the previous artifact/configuration. Test external requests and key journeys, then observe sustained recovery before ending the incident. Add a deployment check for the missed failure.

### S2. New AKS Pods show ImagePullBackOff after ACR access changes. What do you do?
**Answer:** Read Pod events to confirm authorization rather than guessing. Verify the exact repository/digest exists, identify the node's image-pull identity, and check its required registry permissions under the registry's configured authorization mode. Inspect ACR firewall/private DNS and node egress if networking changed. Restore the necessary scoped access, confirm new Pods can pull, and verify rollout readiness. Application workload identity is not automatically the node's pull identity.

### S3. A Linux application slows down and then dies nightly. How do you find the cause?
**Answer:** Build a timeline around the failures using application logs, journal/kernel messages, scheduled jobs, backups, and resource metrics. Check OOM events, disk/inode exhaustion, I/O latency, log rotation, and recurring load. Correlate a suspected job with resource pressure and validate the mechanism before changing it. Adjust scheduling, concurrency, retention, capacity, or application behavior as justified, then verify the next relevant nightly cycle.

### S4. Terraform is locked, a pipeline failed midway, and the team needs an urgent fix. What is the safe sequence?
**Answer:** Identify the active operation and stop competing deployment attempts. Confirm whether any process still owns the lock; capture its error and inspect partial changes. Recover a genuinely stale lock through the documented procedure, then verify backend/workspace and generate a fresh plan. Apply only the reviewed corrective change and verify infrastructure/application health. Record emergency changes so later normal runs do not undo them unexpectedly.
