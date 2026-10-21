# Day 16: Azure Security and DevSecOps — Interview Questions and Answers

## Interview questions

### Q1. What does DevSecOps mean in practice?
**Answer:** Integrate security into design, source review, builds, deployment, and operation with clear ownership and actionable feedback. Use threat modeling, dependency/secret scanning, secure identities, policy, and runtime detection. A pipeline full of tools is insufficient if findings are ignored or exceptions have no owners and expiry.

### Q2. What is Azure Key Vault used for?
**Answer:** Key Vault manages secrets, cryptographic keys, and certificates with controlled access and auditing. Use workload identity where supported to retrieve needed values. Plan networking, access permissions, rotation, soft-delete/recovery settings, and availability. Do not place a vault credential in source code to retrieve another credential.

### Q3. How should Azure RBAC be designed?
**Answer:** Assign the narrowest suitable role at the smallest practical scope to a managed group or workload identity. Separate deployment and application data-access permissions. Review inherited access and privileged assignments regularly, use time-limited elevation where appropriate, and validate the exact operation rather than assuming broad role names are safe.

### Q4. How does workload identity federation reduce secret risk?
**Answer:** A trusted workload presents an issued token that is exchanged for a target identity's authorized access, avoiding a stored long-lived client secret. Trust requires correct issuer, subject, and audience matching. The identity's roles and the pipeline's authorization still determine what the workload can actually do. See [Azure Pipelines federation](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/configure-workload-identity?view=azure-devops).

### Q5. How do SAST, DAST, and software composition analysis differ?
**Answer:** SAST analyzes source or compiled representations without exercising the deployed application. DAST tests a running application from an external perspective. Software composition analysis inspects third-party dependencies and associated risks. Each has coverage gaps, so select complementary checks according to the application's threat model.

### Q6. What does SonarQube contribute?
**Answer:** SonarQube analyzes code quality and supported security issues and can enforce configured quality gates. Exact capabilities depend on language, edition, and configuration. Review the findings and gate result, and distinguish a completed scan from a passed policy. It does not replace dependency or runtime security assessment.

### Q7. What can Trivy scan?
**Answer:** Depending on the selected scan mode and configuration, Trivy can detect vulnerabilities, exposed secrets, and infrastructure misconfigurations across supported targets. Pin and maintain the scanner and its databases, define meaningful failure criteria, and assess findings in context. A clean scan only reflects the scanner's coverage and known information.

### Q8. What does Microsoft Defender for Cloud provide?
**Answer:** It offers security posture and supported workload-protection capabilities across configured resources and plans. Coverage, telemetry, permissions, and cost depend on enabled features. Findings require triage and remediation ownership; enabling a product does not automatically secure every workload or replace application-specific controls.

### Q9. How should secrets be rotated?
**Answer:** Identify consumers, introduce the new credential with overlap when supported, update consumers, verify they use it, and revoke the old credential. Account for caches, long-running processes, failover environments, and rollback. Monitor authentication errors during rotation and avoid logging either credential.

### Q10. What is an SBOM?
**Answer:** A software bill of materials inventories components in a software artifact, often including package versions and identifiers. It supports vulnerability investigation and supply-chain traceability. Bind it to the actual built artifact and retain it with provenance; an SBOM does not itself prove components are safe.

### Q11. How do you protect a software supply chain?
**Answer:** Protect source and build permissions, pin and verify dependencies, isolate trusted build contexts, scan artifacts, and retain provenance. Use signing and admission verification when required. Deployment must verify a trusted artifact identity rather than accepting any image with a familiar mutable tag.

### Q12. How should a security exception be handled?
**Answer:** Document the exact finding and affected artifact, exploitability assessment, compensating controls, accountable owner, approval, and expiry. Scope the exception narrowly and track remediation. A blanket scanner disablement removes visibility and makes later releases inherit risk that was never assessed.

## Scenario-based questions

### S1. A critical vulnerability is found just before release. What do you do?
**Answer:** Identify the affected component and artifact, whether vulnerable code is reachable, exposure, available fixes, and active exploitation evidence. Apply the established release policy and involve the responsible security/application owners. Prefer patching or replacing the component and rebuilding/retesting. If an exception is justified, record scope, compensating controls, owner, and expiry; verify production uses only the approved artifact.

### S2. A federated service connection fails token exchange. How do you investigate?
**Answer:** Inspect the sanitized error and compare configured issuer, subject, and audience with the actual connection's expected values. Check tenant, application/managed identity, federation credential, token timing, and pipeline authorization. After authentication works, validate Azure roles separately. Use values produced by the current connection setup rather than assuming a historical issuer format or replacing federation with a permanent secret.

### S3. A secret appears in a pipeline log. What is the incident response?
**Answer:** Revoke or rotate the exposed value, restrict the affected log and artifact access, and determine who could have retrieved it. Inspect its use and related downstream exposure. Fix the logging path, such as shell tracing or unsafe command arguments, then verify a new run does not disclose credentials. Deleting one log does not invalidate copies or the original secret.

### S4. A pipeline gets permission errors and someone suggests Owner at subscription scope. What would you do?
**Answer:** Capture the denied action, resource scope, and actual identity. Map that operation to an appropriate built-in or carefully defined custom role and check whether the issue is data-plane access or networking instead. Grant the smallest necessary scope through the approved access process and test the operation. Use a separate controlled process for role assignments if the deployment itself should not administer access.
