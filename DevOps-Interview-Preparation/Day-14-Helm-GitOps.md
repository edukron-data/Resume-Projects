# Day 14: Helm and GitOps — Interview Questions and Answers

## Interview questions

### Q1. What is Helm?
**Answer:** Helm packages Kubernetes resources into charts and manages releases of rendered resources. A chart combines metadata, templates, default values, and optional dependencies. It helps reuse deployment configuration, but rendered manifests still need Kubernetes API validation, appropriate RBAC, and application-level verification.

### Q2. What are the main chart files?
**Answer:** `Chart.yaml` describes the chart, `values.yaml` supplies defaults, and `templates/` contains resource templates and helpers. `charts/` can contain dependencies, and `values.schema.json` can validate value structure. Keep chart version and application version distinct because packaging and application changes are not identical.

### Q3. How are Helm values overridden?
**Answer:** Supply values files with `-f` and selected overrides with `--set` or related flags. Later supplied values take precedence for overlapping keys under Helm's merge rules. Keep environment overrides reviewable and avoid passing secrets on command lines where they can enter logs or process listings.

### Q4. What do template, lint, and upgrade do?
**Answer:** `helm template` renders manifests for inspection, `helm lint` checks common chart issues, and `helm upgrade --install` installs or updates a release. Rendering and linting do not prove the API server accepts all resources or that the application will start. Validate against the intended cluster capabilities.

### Q5. What is a Helm release?
**Answer:** A release is an installation of a chart with its values and revision history in a namespace. Release name and namespace identify it in normal Helm usage. `helm list`, `helm status`, `helm history`, and `helm get values` help inspect deployed configuration and prior revisions.

### Q6. How does Helm rollback work?
**Answer:** `helm rollback RELEASE REVISION` reapplies a prior release revision's configuration. Check resources and hooks carefully and validate application health afterward. It does not reverse external database changes or recover deleted business data. Rollback also requires compatible images, secrets, and dependencies to remain available.

### Q7. What are Helm hooks, and what are their risks?
**Answer:** Hooks run resources at defined lifecycle points, often for initialization or migration jobs. Ordering, failure behavior, deletion policies, and repeat execution matter. A hook that modifies a database can outlive the application rollout and make rollback unsafe, so design it for controlled retries and compatibility.

### Q8. What are the core ideas of GitOps?
**Answer:** Desired system state is declared, versioned, reviewed, and reconciled by automation. A controller detects differences between the declared source and the live cluster and applies according to policy. Auditability depends on protecting the repository and controller permissions, not simply storing YAML in Git.

### Q9. What does Argo CD do?
**Answer:** Argo CD compares an application's declared source, including supported Helm or Kustomize configurations, with live Kubernetes state. It reports synchronization and health and can reconcile changes. Authentication, project restrictions, repository trust, sync permissions, and cluster access determine its security boundary.

### Q10. How do auto-sync, self-heal, and prune differ?
**Answer:** Automated sync can apply eligible desired-state changes. Self-heal enables correction of live-state deviations, and prune enables deletion of resources removed from desired state. Treat deletion policy carefully for shared or durable resources and understand empty-application safeguards. See [Argo CD automated sync](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/).

### Q11. How should secrets be handled in GitOps?
**Answer:** Store secret references or approved encrypted representations in Git, with decryption or retrieval restricted to authorized runtime components. Integrations may use a secret manager or encryption controller. Plain base64 Kubernetes Secret data is readable and should not be treated as safely encrypted repository content.

### Q12. How do CI and GitOps CD fit together?
**Answer:** CI tests code, builds and scans an image, and publishes an immutable image digest. A reviewed change updates the environment's deployment declaration to that digest. The GitOps controller reconciles it and reports health. This separates artifact creation from environment promotion while preserving source-to-deployment traceability.

## Scenario-based questions

### S1. A manual kubectl hotfix keeps disappearing. Why?
**Answer:** A reconciler may be restoring the Git-declared state. Inspect Argo CD ownership and sync history, then capture the intended hotfix in the reviewed source so it becomes durable. For an urgent incident, use a documented temporary reconciliation pause if necessary, with an owner and expiry. Restore reconciliation after Git matches the intended live state and verify application health.

### S2. A Helm upgrade leaves Pods unhealthy. What is your response?
**Answer:** Inspect release status/history, rendered values, Pod events, logs, and probe failures. Compare the new manifest, image, and configuration with the last healthy revision. If safe, roll back or correct the chart and redeploy; assess migration hooks and data compatibility first. Verify external requests and controller convergence. A command returning successfully does not prove the release became ready.

### S3. Argo CD says Synced, but the application is broken. How is that possible?
**Answer:** Synced means the compared manifests match desired state, not that all user behavior is correct. Inspect the separate health assessment, Pods, Services, ingress, configuration, and dependencies. Run a real user-journey check and correlate errors. Add suitable health customization and deployment verification so future promotion depends on meaningful behavior as well as declarative convergence.

### S4. A Git change removes a persistent resource. How do you prevent accidental data loss?
**Answer:** Review the rendered diff and the controller's prune policy before sync. Use resource-specific deletion safeguards and appropriate storage reclaim/retention settings, restrict approvals for destructive changes, and maintain tested backups. Understand that preventing deletion can leave intentional drift that requires a managed resolution. If removal is intended, define data migration and recovery criteria before authorizing it.
