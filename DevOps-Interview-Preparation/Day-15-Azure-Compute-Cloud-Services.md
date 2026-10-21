# Day 15: Azure Compute and Cloud Services — Interview Questions and Answers

## Interview questions

### Q1. When would you choose a VM, App Service, or AKS?
**Answer:** Choose VMs for OS-level control or legacy requirements, App Service for managed web/API hosting with less infrastructure operation, and AKS when Kubernetes orchestration and ecosystem capabilities justify its complexity. Evaluate scaling, networking, compliance, deployment model, team skills, and total operating cost.

### Q2. What is Virtual Machine Scale Sets?
**Answer:** VMSS manages a group of VM instances with a shared model and supported scaling/orchestration behavior. It supports workloads that can spread across instances and appropriate load balancing. Instances should be replaceable; externalize durable data and choose upgrade/repair policies suited to the application.

### Q3. What is the role of an App Service plan?
**Answer:** The plan defines the underlying compute capacity and supported capabilities for hosted apps. Apps sharing a plan can contend for its resources. Scaling the plan changes available instances or SKU according to supported options. Evaluate isolation requirements instead of assuming each app has dedicated compute.

### Q4. What are the main Azure Storage services?
**Answer:** Blob Storage stores objects, Azure Files provides managed file shares, Queue Storage supports message-based decoupling, and Table Storage stores key/attribute data. Choose according to access pattern and semantics. Storage account networking, redundancy, identity authorization, and data-protection settings need separate configuration.

### Q5. How do system-assigned and user-assigned managed identities differ?
**Answer:** A system-assigned identity is tied to the lifecycle of its Azure resource. A user-assigned identity is a separate resource that can be attached to supported workloads and managed independently. Both still need explicit permissions to target resources; having an identity is not an authorization grant.

### Q6. How does Azure SQL differ from SQL Server on a VM?
**Answer:** Azure SQL Database provides a managed database service with a service-specific operational model. SQL Server on a VM gives more OS and instance control while leaving more administration to the team. Compare compatibility, cross-database needs, networking, availability, licensing, and recovery requirements before choosing.

### Q7. How should Azure CLI authentication work in automation?
**Answer:** Prefer a supported workload identity federation or managed identity flow, scoped to required resources, rather than an interactive personal login. Explicitly select and validate the intended tenant and subscription. Avoid printing tokens or relying on a developer's cached CLI session on a shared build agent.

### Q8. How does autoscaling differ from scaling up?
**Answer:** Scaling up changes instance size or service tier; scaling out changes instance count. Autoscaling adjusts supported capacity according to metrics or schedules. Set meaningful thresholds, cooldowns, minimums, maximums, and budgets. Stateful bottlenecks and downstream limits can prevent extra instances from improving throughput.

### Q9. What do availability, backup, and disaster recovery solve?
**Answer:** Availability reduces interruption from component failures. Backup provides recoverable data copies or points in time. Disaster recovery restores service after larger failures according to recovery time and recovery point objectives. Replication can replicate accidental deletion, so high availability is not a substitute for backup.

### Q10. How do you deploy cloud resources consistently?
**Answer:** Use reviewed IaC, environment-specific inputs, scoped identities, and a controlled pipeline. Validate planned changes and policy requirements, deploy with clear ownership, and verify outputs and application connectivity. Use tags for operational ownership and cost reporting, while recognizing tags alone do not enforce access boundaries.

### Q11. What is the difference between control-plane and data-plane permissions?
**Answer:** Control-plane permissions manage resources through Azure Resource Manager. Data-plane permissions govern operations on the service's content, such as reading blobs or querying a database. A management role does not necessarily grant data access; authorization requirements vary by service and configured access model.

### Q12. How would you control Azure costs?
**Answer:** Measure actual utilization and unit costs, right-size compute, schedule nonproduction capacity, review storage retention, and remove confirmed unused resources. Apply budgets and cost alerts, and evaluate savings options against usage commitments. Keep resilience requirements visible so a cost reduction does not silently weaken recovery or availability.

## Scenario-based questions

### S1. An App Service cannot reach a database with a private endpoint. What do you inspect?
**Answer:** Confirm the app has the required outbound VNet integration and that DNS resolves the database hostname privately from the app environment. Check routes, security controls, private endpoint status, and database authentication. An inbound private endpoint on the web app does not itself provide outbound VNet integration. Validate connectivity from the running application and execute a representative database request.

### S2. VMSS adds instances, but requests still fail under load. Why?
**Answer:** Check whether new instances complete initialization, pass load-balancer health probes, and receive traffic. Inspect autoscale timing, quota, maximum capacity, application readiness, and shared dependencies such as the database. Compare per-instance and backend metrics. Fix the bottleneck or readiness delay, then repeat a controlled load test; more compute cannot repair an exhausted downstream connection pool.

### S3. An application using managed identity gets 403 from Blob Storage. What would you check?
**Answer:** Identify the actual runtime identity and the requested operation. Verify a suitable blob data role at the correct scope, role propagation, and storage network restrictions. Distinguish identity failures from firewall or private DNS problems using service diagnostics. Avoid granting broad subscription ownership; assign the required data access and confirm the specific blob operation succeeds from the workload.

### S4. A region becomes unavailable. How would you restore service?
**Answer:** Follow a tested recovery plan tied to the agreed RTO/RPO. Confirm the scope, activate or deploy secondary compute, assess database replication and potential data loss, and update traffic routing after health validation. Verify identities, secrets, DNS, and private dependencies in the recovery region. Communicate measured recovery status and plan controlled failback after the primary region stabilizes.
