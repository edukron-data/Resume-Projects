# Day 13: Advanced Terraform — Interview Questions and Answers

## Interview questions

### Q1. Why does state locking matter?
**Answer:** Locking prevents concurrent writers from corrupting or conflicting over one state. Support and implementation depend on the backend. When lock acquisition fails, investigate the current owner rather than bypassing protection. A lock does not replace review or protect against separate states accidentally managing the same object. See [state locking](https://docs.hashicorp.com/terraform/language/state/locking).

### Q2. What does terraform import do?
**Answer:** Import associates an existing remote object with a Terraform resource address. The traditional CLI command imports into state; configuration must also describe the object. Declarative import blocks support reviewable import workflows. Avoid binding one remote object to multiple resource addresses. See [CLI import](https://developer.hashicorp.com/terraform/cli/import).

### Q3. What is infrastructure drift?
**Answer:** Drift occurs when remote infrastructure differs from its declared or recorded expectations, often due to manual changes or external automation. A normal refreshed plan can expose differences. Decide whether the configuration should restore the intended setting or be updated to accept an authorized change.

### Q4. What is refresh-only mode?
**Answer:** `terraform plan -refresh-only` proposes updates to state and root outputs based on observed remote changes without proposing configuration-driven infrastructure changes. Applying a refresh-only plan accepts those observations into state. It does not modify configuration or mean the infrastructure now complies with the declared design.

### Q5. Compare count and for_each.
**Answer:** `count` identifies instances by numerical index and suits homogeneous quantities. `for_each` identifies instances by map keys or set elements and suits stable identities. Removing a list entry with `count` can shift later indices. Choose stable keys and plan address migrations when changing the instance model.

### Q6. Explain lifecycle settings.
**Answer:** `create_before_destroy` changes replacement ordering when supported by the resource constraints. `prevent_destroy` rejects certain destructive plans while the lifecycle rule remains in configuration. `ignore_changes` ignores specified update differences. None is a complete data-protection strategy; name uniqueness, quotas, and removed configuration can change their practical effect.

### Q7. What are CLI workspaces, and when are they insufficient?
**Answer:** CLI workspaces select separate state instances for the same configuration and backend setup. They do not automatically separate credentials, permissions, or subscriptions. Strong environment isolation may require separate root configurations, backends, identities, and pipelines. HCP Terraform workspaces have a broader meaning. See [workspace management](https://developer.hashicorp.com/terraform/cli/workspaces).

### Q8. How do you move a resource into a module without recreating it?
**Answer:** Keep remote identity and relevant arguments unchanged, and declare a `moved` block from the old address to the new address. Review a plan to verify an address move rather than replacement. A carefully controlled `terraform state mv` can also migrate addresses, but modifies state directly.

### Q9. When should depends_on be used?
**Answer:** Use it for a real dependency that cannot be expressed through data references, such as a required policy attachment before another operation. Prefer implicit references where possible. Explicit dependencies only order Terraform operations; API completion may still precede eventual permission propagation or application readiness.

### Q10. How do you design reusable modules?
**Answer:** Keep a focused responsibility, validate inputs, expose useful outputs, document identity and networking assumptions, and version releases. Test realistic compositions and upgrade paths. Avoid embedding environment credentials, subscription-specific values, or an excessive number of switches that turn one module into several unrelated designs.

### Q11. How do you control provider and module upgrades?
**Answer:** Set compatible version constraints, commit provider selections in the dependency lockfile, and pin module versions or source revisions. Upgrade intentionally in a branch, inspect the plan and release notes, and test representative environments. The provider lockfile does not lock remote module versions; module sources need their own pinning.

### Q12. How would you secure Terraform execution in CI/CD?
**Answer:** Separate validation, plan, review, and apply as appropriate; use scoped federated identities and protected execution resources. Secure state and plan artifacts, serialize writes, and ensure the applied saved plan corresponds to the reviewed change. Do not run untrusted PR code with production state or credentials.

## Scenario-based questions

### S1. A failed run leaves a state lock. How do you recover?
**Answer:** Identify the lock owner, operation, timestamp, and associated pipeline. Confirm no process is still applying and stop competing retries. If the lock is genuinely stale, use the backend-appropriate recovery procedure or `terraform force-unlock LOCK_ID` with the exact lock identifier. Then inspect state and a fresh plan before proceeding. Never disable locking to race an active apply.

### S2. Someone manually changes an NSG rule in production. How do you reconcile it?
**Answer:** Determine whether it was an authorized incident mitigation and assess its security impact. Generate a plan to see the exact difference. Either update reviewed code to preserve the approved rule or apply the declared configuration to restore the intended rule. Verify connectivity and protection afterward, and address why normal change controls were bypassed. Refreshing state alone does not codify the accepted change.

### S3. A state file is lost but Azure resources still exist. What do you do?
**Answer:** Freeze writes and confirm the correct backend, key, and workspace before concluding state is lost. Prefer restoring a verified backend version with the correct lineage and recovery process. If no usable backup exists, inventory actual resources and carefully import them into matching configuration. Review plans for unwanted creates/deletes before any apply. Recreating everything can destroy data or create ownership conflicts.

### S4. Changing a list causes Terraform to replace several unrelated resources. Why?
**Answer:** The configuration may use `count`, so deleting or reordering an entry changes index-to-object relationships. Compare old/new addresses and planned argument changes. Introduce stable `for_each` keys and use explicit moved mappings or controlled state moves for each existing instance. Validate that the resulting plan preserves remote objects and changes only the intended resource before applying.
