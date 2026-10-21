# Day 12: Terraform Fundamentals — Interview Questions and Answers

## Interview questions

### Q1. What is infrastructure as code?
**Answer:**

IaC expresses infrastructure configuration in files that can be versioned, reviewed,  
validated, and automated. Terraform uses declarative configuration: describe the desired  
resources and let it plan actions through providers. Version control improves  
traceability but does not replace secure credentials, change review, or recovery planning.

For example, a reviewed Terraform change can create the same networking pattern  
across environments using different inputs. The code becomes a shared description of  
intent. I still inspect the plan and actual service behavior because a syntactically  
valid configuration can create an unsuitable or insecure design consistently.

### Q2. What are providers, resources, and data sources?
**Answer:**

A provider implements interaction with an API such as Azure. A resource manages  
an object's lifecycle. A data source reads information for use in configuration  
without necessarily managing that object's lifecycle. Pin compatible provider  
versions and commit the dependency lockfile for reproducible provider selection.

For example, an Azure resource group can be managed as a resource, while an  
existing shared subnet may be read through a data source. Ownership should  
be explicit so two teams do not both manage the same object. Provider  
authentication, version behavior, and API permissions affect both reads and writes.

### Q3. How do variables, locals, and outputs differ?
**Answer:**

Input variables parameterize a module, locals name reusable expressions  
inside it, and outputs expose selected results. Add type constraints, descriptions,  
and validation to meaningful inputs. A `sensitive` marking reduces  
routine display but does not guarantee the value is absent from stored state.

I use variables for environment decisions such as location, locals for derived  
naming rules, and outputs for IDs needed by callers. Clear boundaries make modules  
easier to understand. Secret inputs require secure execution and storage because their  
values may still appear in state or saved plans despite redacted console output.

### Q4. What is a Terraform module?
**Answer:**

A module is a collection of Terraform configuration files treated as a unit. The  
root module calls child modules to encapsulate reusable infrastructure. Good modules  
have focused responsibilities and clear inputs/outputs. Avoid exposing every  
provider argument or hiding critical security choices behind undocumented defaults.

For example, a network module might create a VNet and subnets and return their IDs  
to an application root module. Its contract should describe required permissions  
and naming constraints. I avoid modules that hide destructive lifecycle choices  
or combine unrelated resources that different teams need to change independently.

### Q5. What is Terraform state used for?
**Answer:**

State records the relationship between configured resource addresses and managed remote  
objects and stores metadata used for planning. It can contain sensitive values. Protect  
access, use an appropriate remote backend for team workflows, and maintain recoverable  
versions. See [Terraform state](https://developer.hashicorp.com/terraform/language/state).

If state points to the wrong object or disappears, Terraform may propose unexpected  
creates, updates, or deletions. I therefore verify the backend and environment before  
interpreting a surprising plan. State recovery is an operational capability with access  
controls and tested procedures, not simply copying a JSON file into source control.

### Q6. What does terraform init do?
**Answer:**

`terraform init` initializes the working directory, configures the backend,  
retrieves modules, and installs selected providers. It does not provision application  
infrastructure by itself. Backend migration options require care because pointing  
at the wrong backend can make Terraform appear to have no existing resources.

I run initialization after selecting the intended configuration and backend  
inputs, especially in CI. A successful init proves initialization completed, not  
that the target infrastructure is correct. After backend or dependency changes,  
I review the lockfile and a fresh plan before allowing any resource mutation.

### Q7. What does terraform plan do?
**Answer:**

A plan compares configuration and refreshed resource information with  
state to propose changes. Review creates, updates, replacements, and  
deletions. `terraform plan -out=tfplan` saves an execution plan for later  
application; protect that file because it can contain sensitive values.

I focus review on why each action is required, especially replacement triggers  
and deletion of data-bearing resources. Unknown values are not automatically errors  
but may hide details until apply. A saved plan should be tied to the reviewed  
source and environment and reapplied only under the intended controlled workflow.

### Q8. What happens during terraform apply?
**Answer:**

Terraform executes planned API operations according to dependencies and  
records results in state. An apply can partially succeed, so a failure  
does not mean nothing changed. Inspect the error, state, and actual  
resources, then produce a fresh plan before deciding how to continue.

For example, networking may be created successfully before a later resource fails  
due to quota. Repeating the original expectation that nothing exists can then  
cause confusion. Terraform normally tracks completed actions, but I reconcile  
ambiguous API results carefully and use a new plan to identify the safe next steps.

### Q9. What does terraform destroy do?
**Answer:**

It plans and applies destruction of managed resources for the selected  
configuration/state context. It is not a cleanup command to run casually after an  
error. Confirm environment, backend, dependencies, backups, and data implications;  
some resources may contain durable business data or serve other systems.

Before an intentional environment teardown, I check external consumers and data retention  
requirements as well as the selected state. A resource's presence in Terraform  
does not prove it is disposable. I review a destroy plan and define verification afterward,  
including any resources intentionally retained by service or lifecycle settings.

### Q10. Why use a remote backend?
**Answer:**

A suitable remote backend centralizes state with controlled access and can  
support locking and recovery capabilities. For Azure Blob state, configure the  
`azurerm` backend and its identity/storage permissions. Backend capabilities vary;  
remote state is not automatically secure merely because it is off the laptop.

For a shared Azure deployment, the backend identity needs suitable access to its storage,  
which is separate from application deployment permissions. I configure protected  
state access and recoverable versions intentionally. Network restrictions must also allow  
the authorized execution environment to reach the backend without exposing it broadly.

### Q11. How does Terraform determine dependencies?
**Answer:**

References such as a subnet using a VNet resource ID create implicit  
dependencies. `depends_on` expresses hidden ordering requirements that  
references do not capture. Avoid unnecessary broad dependencies because  
they reduce parallelism and can make more values unknown during planning.

For example, referencing a subnet ID naturally orders creation after that subnet. I use  
explicit dependencies only when the real prerequisite is otherwise invisible. Terraform  
ordering does not guarantee application readiness, so runtime checks or service-specific  
waiting may still be required before subsequent deployment operations can succeed.

### Q12. What belongs in a Terraform repository?
**Answer:**

Commit configuration, module source or version references, documentation, and  
`.terraform.lock.hcl`. Exclude `.terraform/`, local state files, saved plans,  
credentials, and secret variable files. Validate formatting and configuration  
in CI, and authorize plan/apply identities with only the permissions required.

I keep example variable files free of secrets and explain how users supply protected  
values in their environment. Validation should catch basic mistakes before an apply  
reaches Azure. A clear repository layout also separates reusable module changes  
from environment composition so reviewers can understand the scope and blast radius.

## Scenario-based questions

### S1. Two engineers need to manage the same Azure infrastructure. How do you organize state?
**Answer:**

Move the intended shared state into a protected backend with locking support and recoverable versions.  
Use distinct state scopes for independently managed environments, scoped identities, and a controlled  
apply workflow. Migrate existing state rather than starting empty. Verify a postmigration plan shows the  
expected resources and changes, then remove obsolete local copies according to handling requirements.

I confirm that every engineer and pipeline uses the same intended state scope instead  
of keeping competing local copies. Existing state migration should be controlled and  
recoverable. Afterward, a no-surprise plan is important evidence that remote objects remain  
associated correctly and were not accidentally treated as unmanaged infrastructure.

### S2. Terraform proposes deleting a production resource unexpectedly. What do you do?
**Answer:**

Stop before apply and inspect the resource address, configuration diff, selected workspace/backend,  
provider change, and any immutable argument change. Determine whether the object moved between modules or  
was removed accidentally. Use a reviewed address migration when appropriate, correct the configuration,  
and regenerate the plan. Do not approve deletion based only on a pipeline's green validation result.

If the proposed deletion results from a renamed address, I migrate the address relationship  
instead of deleting and recreating the remote object. If an immutable property  
changed intentionally, I assess downtime, quota, and data consequences. The decision  
should explain the actual replacement mechanism, not simply label the plan unexpected.

### S3. Apply fails after creating some resources. Should you rerun it immediately?
**Answer:**

First inspect the failing API operation, existing state, and newly created resources. Correct the  
cause, such as quota, permissions, or invalid configuration, and generate a new plan. If an object  
exists but is missing from state, reconcile ownership carefully rather than creating a duplicate.  
Apply the reviewed remaining changes and confirm final infrastructure and application health.

I also check whether the failed API operation might have completed remotely despite a client  
error. That ambiguous case may require import or provider-specific recovery rather than  
another create. Once reconciled, I verify outputs and dependent connectivity so the final  
success reflects working infrastructure rather than only an error-free Terraform command.

### S4. An output is marked sensitive. Is it safe to commit state or a saved plan?
**Answer:**

No. Sensitive display marking does not make state or plans safe for source control. Restrict these  
artifacts to protected storage, avoid publishing unrestricted CI artifacts, and use secure identity/secret  
handling. If exposure already occurred, determine which credentials were included and rotate them  
as needed. Check repository history and pipeline retention paths, not only the current working tree.

I examine all places the value may have traveled, including local backups, uploaded plan  
files, diagnostic logs, and developer clones. Access restrictions and encryption reduce exposure  
but do not justify committing sensitive artifacts. Recovery includes replacing exposed  
credentials and correcting the process that published them, not just deleting the latest file.
