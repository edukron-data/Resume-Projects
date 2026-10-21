# Day 6: Azure DevOps Fundamentals — Interview Questions and Answers

## Interview questions

### Q1. What services make up Azure DevOps?
**Answer:**

Boards tracks work, Repos hosts source control, Pipelines runs CI/CD,  
Artifacts hosts packages, and Test Plans supports test management and  
exploratory/manual testing. These services can link a requirement to a commit,  
build, test result, and deployment, improving traceability across delivery.

For example, a bug can be tracked in Boards, fixed through a Repos PR, validated  
in Pipelines, and associated with a deployed artifact. This chain helps  
answer what changed and why. The services can also integrate with external  
tools, so adoption does not require every team activity to use one interface.

### Q2. How do organizations and projects differ?
**Answer:**

An organization groups users, projects, and organization-level settings. A  
project contains related repositories, boards, pipelines, and other delivery  
resources. Choose boundaries according to ownership, collaboration, security, and  
administration rather than creating a new organization for every application.

I first identify which teams need shared administration and which resources require isolation.  
A project boundary changes collaboration and permissions, while an organization also  
affects broader administration. Too many boundaries can duplicate configuration; too few can  
make access ownership unclear. The design should match actual team and security requirements.

### Q3. How do Azure Boards work items relate?
**Answer:**

Work-item types and hierarchies depend on the selected process. Common  
structures connect epics, features, backlog items or user stories, tasks, and bugs.  
Area paths organize ownership; iteration paths organize timeboxes. Links  
to PRs and builds show how implementation relates to the tracked requirement.

A work item should have clear acceptance criteria and enough context for  
implementation and verification. I use the board to expose blocked work and delivery  
flow rather than only listing assignments. Linking code and tests back to  
requirements makes release discussions and defect investigations more concrete.

### Q4. What is a service connection?
**Answer:**

A service connection defines how a pipeline authenticates to an external  
resource such as Azure or a container registry. Its Azure permissions and its  
pipeline-use permissions are separate controls. Prefer a supported federated  
identity over stored client secrets and scope access to the resources required.

For example, a deployment pipeline may be authorized to use a connection whose  
Azure identity can modify only one resource group. Both checks must succeed. I  
document the connection's owner and purpose, review access periodically, and avoid  
granting every pipeline automatic use of a privileged production connection.

### Q5. What is an agent pool?
**Answer:**

An agent executes pipeline jobs; a pool groups agents available for scheduling.  
Agent capabilities and job demands influence selection. Pool access does  
not automatically grant access to Azure resources. Avoid placing untrusted  
code on privileged agents that have production connectivity or credentials.

When a job cannot start, I inspect pool permissions, agent availability, and  
capability matching separately. A pool can contain machines with very different  
tools or network paths. I keep agent responsibilities explicit so workloads do  
not accidentally depend on whichever machine happens to accept the next job.

### Q6. Compare Microsoft-hosted and self-hosted agents.
**Answer:**

Microsoft-hosted jobs receive managed, fresh execution environments with predefined  
tools. Self-hosted agents provide control over networking, installed software,  
and hardware but require patching, cleanup, isolation, and capacity management.  
Private endpoints often require an agent with an appropriate private network path.

For a standard public build, hosted agents can reduce maintenance and avoid residue  
between jobs. For a private deployment, a self-hosted agent may provide the  
necessary connectivity. I compare lifecycle cost and trust requirements, including  
how the machine is reset and how secrets are kept away from untrusted builds.

### Q7. How do Azure DevOps permissions work?
**Answer:**

Permissions apply through users, groups, resource scopes, and inheritance. Access levels  
and licensing determine available capabilities, while permissions govern authorized  
actions. Repository, pipeline, pool, and environment controls are distinct. Diagnose effective  
permissions, including explicit denies, instead of adding broad administrator roles.

I troubleshoot access using the actual identity and resource scope instead of changing  
several group memberships at once. An explicit deny or restricted pipeline resource  
can explain why a broad membership does not help. After correction, I test the  
required action and confirm that unrelated administrative capabilities were not added.

### Q8. What does Azure Artifacts store?
**Answer:**

Azure Artifacts hosts package feeds for supported ecosystems and Universal  
Packages. A pipeline artifact stores files produced by a pipeline run. Use package  
feeds for versioned dependency consumption and pipeline artifacts for  
run-associated outputs, applying suitable retention and access controls to both.

For example, a reusable internal library belongs in a package feed with an  
intentional version, while a release bundle can be attached to its build run. Consumers  
should request an identified version. I also consider retention because deleting  
the only release output can make later redeployment or recovery impossible.

### Q9. What is Azure Test Plans used for?
**Answer:**

Test Plans organizes test plans, suites, cases, manual execution, exploratory  
testing, and requirement traceability. Automated test results can  
also be published through pipelines. A successful build alone is not proof  
that acceptance criteria or manual compliance checks have been satisfied.

For an acceptance test, I connect the test case to the requirement and preserve  
the execution outcome and evidence. Automated results complement that record  
when the test can be automated reliably. The goal is visible coverage and  
repeatability, not simply increasing the number of test cases stored in the system.

### Q10. How do environments differ from variable groups?
**Answer:**

An environment represents a deployment destination and can provide deployment  
history and resource checks. A variable group shares configuration values and, when  
configured appropriately, secrets across pipelines. Declaring an environment does  
not provision the actual Azure application or automatically isolate its network.

For example, the production environment can require approval while its variable group  
supplies approved configuration to authorized pipelines. Access to either resource  
should reflect its sensitivity. I avoid storing production secrets in a broadly shared  
group just because the deployment destination itself has a protected environment name.

### Q11. What is the difference between a PAT and workload identity federation?
**Answer:**

A personal access token grants Azure DevOps API access within its configured scope  
and lifetime. Workload identity federation exchanges a trusted workload token for  
access to a target service, avoiding a stored long-lived application secret. They solve  
different authentication needs and are not interchangeable in every integration.

A PAT should be narrowly scoped, short-lived where practical, and handled like a  
password. Federation reduces secret rotation work for supported workload authentication  
but still needs a correct trust relationship. I choose the mechanism required  
by the integration rather than assuming one identity method works for every API.

### Q12. How would you structure Dev, QA, and Prod access?
**Answer:**

Use separate deployment identities and appropriately isolated resources or subscriptions,  
protect production environments and service connections, and limit who can approve or  
change controls. Share reviewed templates while keeping environment configuration separate.  
Ensure build authors cannot silently gain production access by changing pipeline YAML.

The same pipeline template can support several environments while their identities and  
resource permissions remain distinct. This limits the impact of a development error  
or compromised build. I verify that a contributor cannot redirect production credentials  
into an arbitrary script by changing an unprotected variable or pipeline reference.

## Scenario-based questions

### S1. A pipeline cannot use a valid service connection. What would you check?
**Answer:**

Distinguish pipeline authorization from target-resource authorization. Confirm the pipeline is allowed  
to use that service connection, the connection belongs to the expected project, and its checks are  
satisfied. Then validate federation configuration or credential validity and Azure RBAC scope. Check the exact  
failing operation, fix only the missing permission, and rerun a limited validation before deployment.

I read the failing task's sanitized error to establish whether access failed before  
reaching Azure or during a particular Azure operation. A connection validation test may not  
exercise every deployment action. Verification should therefore include the actual required  
scope and operation without exposing tokens or granting broad temporary permissions.

### S2. Jobs remain queued even though a self-hosted agent is online. Why?
**Answer:**

Check available parallel-job capacity, whether the agent is already busy, the selected pool,  
pool-use permissions, and matching demands/capabilities. Inspect agent diagnostics and  
organization/project settings. An online agent may still be unsuitable for that job. Remove an incorrect  
demand or add an appropriately configured agent, then confirm the queued job actually starts.

If capabilities do not match, the scheduler cannot assign the job even though  
the machine appears healthy. If parallel capacity is exhausted, adding tools to  
the agent will not help. I distinguish these causes before changing configuration  
and confirm the resolved agent has the network access required by later tasks.

### S3. A developer can edit YAML and wants to bypass production review. How do you prevent that?
**Answer:**

Put approvals and checks on protected resources controlled by authorized owners, restrict production  
service-connection access, and protect the relevant repository branches. Ensure production credentials are  
unavailable through an unprotected alternative path. Review bypass permissions and audit configuration changes.  
Validate the design using a harmless pipeline that attempts unauthorized production-resource access.

I also check alternative pipelines, direct deployment tools, and human credentials because  
controls must cover every production path. An approval boundary is weak if authors can  
reach the same target through another unrestricted connection. Regular permission review  
and harmless access tests help demonstrate that the intended separation actually works.

### S4. A new team needs a project setup. What would you establish first?
**Answer:**

Identify owners and required access, select the work-item process, create repositories and branch  
policies, configure agent pools and scoped service connections, and establish environment checks.  
Add a minimal pipeline that builds, tests, and publishes an identifiable artifact. Link a  
sample work item through PR and deployment, and document routine ownership and recovery procedures.

I would also establish naming, retention, ownership, and incident contacts so the project  
remains maintainable after the initial setup. The first pipeline should prove an  
end-to-end delivery path with minimal complexity. Additional templates and controls can  
then be added according to demonstrated needs rather than copied without understanding.

Identity reference: [Azure Resource Manager service connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops).
