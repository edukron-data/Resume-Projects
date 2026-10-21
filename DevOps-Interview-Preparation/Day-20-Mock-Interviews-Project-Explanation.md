# Day 20: Mock Interviews and Project Explanation — Questions and Model Answers

## Interview questions

### Q1. Tell me about yourself as an Azure DevOps candidate.
**Answer:** "My background is in [your actual role or training], with practical work in [tools you have used]. In [a real project or lab], I handled [your contribution] and validated it through [evidence]. I am strongest in [two relevant areas] and am developing [one area]. I am interested in this role because [specific match]." Replace brackets with facts; identify labs as labs.

### Q2. Explain an end-to-end Azure DevOps project architecture.
**Answer:** "In an example architecture, Azure Repos holds application code and Terraform. PR validation runs tests and security checks. CI publishes a versioned image to ACR. Terraform provisions the required Azure infrastructure. CD promotes the same image digest through protected environments to AKS using reviewed Helm configuration or GitOps. Workload identities access Key Vault and Azure services, while Azure Monitor and application telemetry verify release health." Adapt this to your real implementation.

### Q3. Explain your CI/CD pipeline in two minutes.
**Answer:** "A change starts with a short-lived branch and PR. Required validation must pass before merge. The main build tests, scans, and packages one artifact with source/run identity. That artifact moves through environment deployments with configuration kept separate. Protected production checks control release, and smoke tests plus telemetry validate it. A known-good artifact and compatible database strategy support recovery." Name your actual tasks and tools when used.

### Q4. Why did you choose AKS instead of App Service or VMs?
**Answer:** "I would choose AKS when Kubernetes scheduling, deployment controls, ecosystem integrations, or workload requirements justify its operating cost. I would prefer App Service for a simpler managed web application and VMs for OS-specific requirements. My decision would compare team skills, networking, scaling, resilience, and cost rather than choosing Kubernetes by default." For a real project, give the actual constraints.

### Q5. How would you design infrastructure for a new application?
**Answer:** Clarify traffic, data sensitivity, availability, RTO/RPO, budget, and team ownership. Choose suitable compute and data services, plan private connectivity and DNS, define identities, and express infrastructure in reviewed modules. Design deployment, monitoring, backup, and recovery with the application team, then validate the design using representative workload and failure tests.

### Q6. How would you explain your Terraform project structure?
**Answer:** "I separate reusable modules from environment root configurations, use typed inputs and clear outputs, and keep each independent state boundary intentional. Remote state has scoped access, locking support, and recovery versions. CI performs formatting/validation and produces a reviewed plan; controlled applies use scoped identity. Provider selections and module versions are managed deliberately." Describe only controls your project actually implements.

### Q7. How do you explain your individual contribution in a team project?
**Answer:** State the team's goal, your specific responsibility, what you changed, and how you verified the result. For example: "The team owned delivery; I implemented [actual pipeline/module], resolved [actual issue], and measured [real evidence]. [Other role] owned [their responsibility]." Distinguish implementation, operation, and observation so the interviewer can assess your actual depth.

### Q8. Describe a production incident you handled.
**Answer:** Use a real Situation–Task–Action–Result example: describe impact, your assigned role, evidence you gathered, mitigation, validation, and preventive follow-up. Explain the mechanism and trade-off rather than listing commands. If you have no production exposure, say so and use a lab failure, clearly stating what was simulated and what production controls would additionally be needed.

### Q9. How do you measure DevOps improvement?
**Answer:** Track delivery speed and stability together: deployment frequency, lead time, failed changes, and service recovery, with explicitly defined measurement rules. Also inspect pipeline duration, developer waiting time, toil, reliability indicators, and cost where relevant. Use trends and context to find bottlenecks rather than treating one metric as a performance target for individuals.

### Q10. What would you do if you disagreed with a teammate's design?
**Answer:** "I would clarify the shared requirements, ask about their constraints, and compare options using evidence such as maintainability, security, reliability, and cost. For an uncertain assumption, I would propose a small experiment. Once an accountable decision is made, I would document the rationale and support implementation, escalating only unresolved risks through the appropriate owner." Add a genuine example if available.

### Q11. How do you prioritize urgent requests from multiple teams?
**Answer:** Assess user impact, security exposure, deadlines, dependencies, and available capacity. Make trade-offs visible and confirm the accountable priority owner. Stabilize active incidents first under the incident process, then schedule durable work and communicate realistic checkpoints. Explain what evidence would change the order rather than promising all requests at once.

### Q12. What would you focus on in your first 90 days?
**Answer:** "Initially I would learn the architecture, deployment process, access model, and incident history. Next I would deliver a small verified improvement while pairing with owners. By the end of the period I would aim to own an agreed delivery or reliability area with documented operation and measurable results." Adapt the scope to seniority and the team's actual priorities.

## Scenario-based questions

### S1. Design a secure delivery platform for a three-tier application. How would you answer?
**Answer:** Start with requirements and choose suitable compute instead of assuming a platform. Describe the request path from TLS ingress to private application and data services, identity-based access, and private DNS. Explain PR checks, one immutable artifact, infrastructure plans, environment promotion, production approvals, and release validation. Finish with monitoring, backups, tested recovery, costs, and the main trade-off. State unknowns rather than inventing capacity numbers.

### S2. A manager asks you to bypass checks to meet a release deadline. What do you say?
**Answer:** "I would explain the specific failed check and likely business impact, then propose the fastest safe route: fix the issue, reduce release scope, or use a documented exception if policy permits. Any exception needs the accountable owner's approval, a bounded scope, compensating controls, and recovery readiness. I would keep stakeholders updated with the evidence and revised release checkpoint." Avoid claiming authority you do not have.

### S3. A release causes an outage while several teams blame one another. How do you lead the response?
**Answer:** Establish an incident lead, focus the group on verified user impact and the change timeline, and assign parallel investigations to named owners. Pause further changes and choose a safe mitigation based on evidence. Give clear updates separating facts from hypotheses and verify recovery through user journeys and telemetry. Afterward, run a blame-free review with owned corrective actions instead of settling fault during the outage.

### S4. The interviewer asks for performance improvements, but you have no measured percentage. How should you answer?
**Answer:** Do not invent one. Explain the observed problem, the change you made, and the evidence you actually retained, such as a before/after pipeline duration, reduced manual steps, or a verified failure correction. State the measurement limitation and describe how you would collect comparable baselines and multiple runs next time. A precise, honest explanation is stronger than an unsupported claim of a large improvement.
