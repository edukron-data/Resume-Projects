# Day 8: Azure CD Pipelines — Interview Questions and Answers

## Interview questions

### Q1. How does continuous delivery differ from continuous deployment?
**Answer:**

Continuous delivery keeps a validated artifact ready for release, with an intentional  
production decision when required. Continuous deployment automatically releases every  
qualifying change to production. Both depend on reliable automation, appropriate  
controls, observability, and recovery; the distinction is the production-release decision.

A team may continuously deliver to a production-ready state while retaining a  
business approval before exposing users to a change. Automation should still  
make that release repeatable. I explain the actual decision point and controls  
rather than calling every pipeline with a deployment task continuous deployment.

### Q2. How would you design a multi-stage YAML pipeline?
**Answer:**

Build and test once, publish an immutable artifact, then deploy it through Dev,  
QA, UAT, and Prod with environment-specific configuration. Express dependencies  
explicitly, validate each deployment, and protect sensitive environments. Record  
artifact identity, configuration revision, and deployment result at every stage.

For example, the Dev stage can run fast integration checks while UAT confirms  
business acceptance and Prod requires protected approval. The same artifact identity  
should remain visible throughout. If a stage changes the package itself, I treat  
that as creating a new artifact that needs corresponding validation and provenance.

### Q3. What are deployment jobs and environments?
**Answer:**

A deployment job models deployment activities and can target an Azure DevOps  
environment. Environments provide deployment history and can carry  
approvals/checks. A normal job can run deployment commands, but a deployment  
job makes deployment tracking and supported lifecycle strategies explicit.

I use the environment's history to connect deployments to pipeline runs and  
outcomes. The job must still execute the correct target-specific deployment commands  
and health checks. Referencing an environment name alone does not create cloud  
resources, configure a network, or prove that the intended application was updated.

### Q4. Where are approvals, checks, and gates configured?
**Answer:**

In YAML pipelines, resource owners configure approvals and checks on protected resources through the service interface,  
separately from YAML. Conditions inside YAML do not replace those controls. Classic release pipelines also have gates  
that evaluate external signals at configured intervals. Distinguish those classic gates from YAML resource checks. See  
[approvals and checks](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops).

A human approval answers whether an authorized person accepts the release, while an  
automated check can evaluate an external signal or policy. I identify who owns each  
control and what happens when it times out. Emergency bypass should follow a documented  
accountable process rather than a YAML change that quietly removes protection.

### Q5. What is artifact promotion?
**Answer:**

Promotion moves the same identified output through release environments after  
validation. Use a fixed package version or image digest and record its build provenance.  
Rebuilding source separately for production can change dependencies or output,  
so it no longer proves that production received the artifact previously tested.

For example, an image digest tested in QA should be the digest selected for  
production, with only approved runtime configuration changing. Mutable tags make this  
harder to prove. I record the artifact and configuration references together so  
an incident investigation can identify exactly what each environment received.

### Q6. Compare rolling, blue-green, and canary deployments.
**Answer:**

Rolling replaces instances gradually. Blue-green maintains separate old and new environments  
and switches traffic. Canary sends limited production traffic to the new version before  
expansion. Each requires health checks and capacity planning; blue-green and canary also  
require suitable traffic routing and compatibility between concurrently running versions.

Rolling updates need enough healthy capacity during replacement, while blue-green typically needs  
duplicate capacity during the transition. Canary adds controlled exposure and comparison rather  
than automatically guaranteeing safety. I discuss session behavior, background processing, data  
compatibility, and routing support because those details often determine whether the strategy works.

### Q7. Are deployment strategies supported identically on every target?
**Answer:**

No. Azure Pipelines deployment-job strategies and lifecycle hooks have target-specific support. For example, its built-in  
rolling strategy targets VM resources. Kubernetes traffic splitting, App Service slot swaps, and other canary/blue-green  
behavior need target-specific tasks, controllers, or routing configuration rather than just a strategy name. See  
[deployment-job support](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs?view=azure-devops).

I verify the selected target and task documentation before promising a particular  
rollout behavior. A lifecycle hook can invoke a command, but traffic splitting still  
depends on the underlying platform. The pipeline should explicitly show how instances  
are updated, how traffic changes, and how failed validation reverses the operation.

### Q8. How do deployment slots help App Service releases?
**Answer:**

A supported App Service plan can provide slots for separate application versions.  
Deploy to a staging slot, warm it, validate it, and swap according to the slot  
configuration. Understand which settings are slot-sticky and ensure staging cannot  
run harmful production background jobs. A slot swap does not undo database changes.

Before swapping, I test the staging hostname and confirm that environment-specific settings  
remain attached to the intended slot. I also consider warm-up, authentication callbacks,  
and scheduled work. After the swap, I validate production requests and retain a compatible  
recovery option rather than assuming staging health guarantees user-facing success.

### Q9. What is a rollback plan?
**Answer:**

Define a known-good artifact, compatible configuration, a traffic or deployment  
reversal method, decision criteria, and verification steps. Database  
changes may require a forward fix or a separately tested data recovery  
procedure. Recovery time is only credible when the process has been exercised.

I define who makes the rollback decision and which measurements trigger it so  
the team does not improvise during an outage. The previous artifact must remain  
available and deployable. I test recovery with realistic dependencies because  
a stored old image is not useful if its secrets or schema are incompatible.

### Q10. What should release validation measure?
**Answer:**

Validate readiness, key user journeys, dependency connectivity, error rate,  
latency, and any required business signals. A successful command or container  
start is insufficient. Compare against a baseline over a meaningful observation  
window and define stop or rollback thresholds before deployment begins.

I choose checks that reflect the application's critical behavior, such as authentication  
and one important transaction, without creating unsafe production test data. Metrics should  
identify the new version separately when possible. The observation window needs enough  
relevant traffic to distinguish a healthy release from one that simply received no requests.

### Q11. How do you prevent concurrent production deployments?
**Answer:**

Use protected-resource exclusive locks and an intentional lock behavior, supported  
by deployment tooling that respects serialization. Ensure every deployment path  
uses the same control. Preventing overlapping infrastructure applies and application  
releases may require separate locks because their shared-state boundaries differ.

An exclusive lock is effective only when all release paths honor the same resource  
boundary. I inspect manual procedures and alternate pipelines as well. The queueing policy  
should intentionally decide whether older waiting releases still run or are superseded,  
because deploying obsolete versions after a newer release can reintroduce defects.

### Q12. How should environment configuration be handled?
**Answer:**

Keep the application artifact stable and supply environment-specific endpoints and nonsecret  
settings separately. Retrieve secrets using authorized identities or protected stores.  
Version configuration where appropriate, validate it before traffic moves, and record  
which configuration accompanied a release so recovery can restore the matching combination.

Configuration should be validated for required keys, permitted values, and target endpoints  
before deployment starts. I avoid baking production-specific settings into the artifact  
that QA tested. When a configuration-only change is released, I still record its  
revision and verify behavior because configuration can cause outages without a code change.

## Scenario-based questions

### S1. Production deployment is successful, but users receive errors. What do you do?
**Answer:**

Pause further promotion, assess user impact, and compare the incident start with the release. Inspect  
application errors, latency, dependency health, and configuration changes. If the new release is  
the likely cause and reversal is safe, restore the prior artifact or traffic target. Verify external  
user journeys and sustained recovery, then investigate why release validation missed the issue.

I preserve the failing version's logs and configuration identifiers before replacement where  
practical. Recovery must be measured from outside the application, including authentication  
and affected workflows. After stabilization, I improve the missed readiness, smoke-test, or  
dependency check so the next release can detect the same failure before broad user impact.

### S2. QA and Prod used the same Git commit but behave differently. Why?
**Answer:**

Compare artifact digests first: the same commit can produce different builds. Then compare configuration,  
secrets, runtime versions, database schema, identity permissions, networking, and feature flags.  
Reproduce the specific failing request. Correct the mismatch and promote the already-validated artifact  
rather than rebuilding blindly; record environment assumptions that should become predeployment checks.

The comparison should include feature flags and identity permissions because those can  
alter behavior even when binaries match exactly. I follow the failing request's dependency  
path rather than comparing every setting without direction. Once the root difference  
is identified, I capture it as code, validation, or documented environment intent.

### S3. A canary has a higher error rate. Should the pipeline continue?
**Answer:**

Pause expansion and compare equivalent traffic with the stable version, checking sample size,  
request types, and dependency conditions. If predefined failure criteria are met, route  
traffic back and capture diagnostics. Do not average the canary into total traffic in a way that  
hides its failures. Resume only after a corrected release meets the agreed health criteria.

I also confirm that the canary cohort is comparable and that errors are  
attributable to the new version rather than a shared outage. If the sample is too  
small, I pause rather than claiming success. The release policy should define  
both failure thresholds and the minimum evidence required to expand traffic.

### S4. A database migration prevents application rollback. How would you avoid this?
**Answer:**

Use an expand-and-contract approach: add backward-compatible schema first, deploy code that works  
during transition, migrate data safely, and remove obsolete schema in a later release. Validate compatibility  
with both old and new application versions. If an incompatible migration already happened,  
assess a forward fix or tested data recovery instead of assuming an image rollback can restore service.

I separate reversible application changes from irreversible data transformations in the release  
plan. Backups need tested restoration and an explicit acceptable data-loss window; their  
existence alone does not make immediate rollback safe. The strongest answer explains compatibility  
sequencing, verification, and who authorizes a potentially disruptive data-recovery operation.
