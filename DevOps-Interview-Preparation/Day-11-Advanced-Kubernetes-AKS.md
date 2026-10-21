# Day 11: Advanced Kubernetes and AKS — Interview Questions and Answers

## Interview questions

### Q1. Compare ClusterIP, NodePort, and LoadBalancer Services.
**Answer:**

ClusterIP exposes a Service within cluster networking. NodePort also exposes a port  
on nodes, subject to network controls. LoadBalancer requests integration with an  
external load-balancing implementation. Service selectors identify backends, and  
EndpointSlices represent them. A Service does not fix unhealthy application containers.

For example, an internal API may need only ClusterIP access while an external  
service uses an appropriate load-balancing path. I check port, target port, selector,  
and ready endpoints independently. A valid Service object can still route no  
traffic when its selector or readiness assumptions do not match the application.

### Q2. How does Ingress differ from a Service?
**Answer:**

A Service provides stable access to selected backend endpoints. An Ingress  
resource declares HTTP(S) routing handled by an installed controller. Creating  
Ingress alone does not install that controller. Controller capabilities, TLS  
configuration, ingress class, and supported APIs determine actual routing behavior.

I identify which controller owns the ingress class and where its external address  
comes from. Then I verify DNS, TLS certificates, host rules, and backend  
ports. An Ingress manifest can be syntactically valid yet unused if no controller  
watches its class or the chosen controller does not support its annotations.

### Q3. How do ConfigMaps and Secrets differ?
**Answer:**

ConfigMaps store nonsecret configuration. Secrets hold sensitive values but base64 encoding  
is not encryption. Restrict API access and configure encryption and external-secret  
integration according to requirements. Environment-variable consumption typically needs Pod  
replacement to see changes; mounted-file updates also require the application to reload them.

Configuration updates need a deliberate rollout or reload strategy so all replicas  
use the intended values. I avoid printing secret content while troubleshooting whether  
it exists. For external secret integrations, I also verify the runtime identity,  
store permissions, network connectivity, and how rotations reach the application.

### Q4. Explain PVs, PVCs, and StorageClasses.
**Answer:**

A PersistentVolume represents storage, a PersistentVolumeClaim requests it, and a  
StorageClass describes provisioning behavior. CSI drivers integrate storage  
systems. Access mode, topology, reclaim policy, and binding mode matter. Persistence  
is not backup; data protection and restore testing are separate responsibilities.

For example, a database claim may require a particular access mode and zone-compatible  
disk. I check reclaim policy before deleting a claim because the underlying  
volume's lifecycle can differ. Backups should be application-consistent where needed,  
and restoration should be tested independently from successful storage provisioning.

### Q5. When would you use a StatefulSet?
**Answer:**

Use StatefulSets when workloads need stable per-Pod identities, ordered lifecycle  
behavior, or per-replica persistent storage. They can support stateful systems but do  
not automatically provide database replication, quorum safety, backups, or  
application-aware failover. Configure those according to the application's own requirements.

For a quorum-based service, stable identities help but availability still depends on  
replica placement, disruption policies, and the software's replication design. I follow  
the application's supported operational model. A StatefulSet controller cannot determine  
whether a database replica is safe to replace or whether a backup is recoverable.

### Q6. What is a DaemonSet?
**Answer:**

A DaemonSet runs a Pod on every eligible node, or a selected subset, and  
adapts as nodes change. It suits node-level log collection, networking, or  
monitoring agents. Eligibility depends on scheduling constraints and  
tolerations; it is not a replacement for a fixed-replica application Deployment.

I scope DaemonSets carefully because a new node can automatically receive their workload  
and resource consumption. Node selectors and tolerations should reflect the intended node  
pools. During upgrades, I verify that monitoring or networking agents become healthy on  
replacement nodes before assuming the rest of the workload is observable and reachable.

### Q7. How does HPA work?
**Answer:**

The Horizontal Pod Autoscaler adjusts workload replicas using configured metrics. CPU  
utilization is measured relative to resource requests and needs the relevant metrics  
pipeline. Custom metrics require an adapter or supported integration. HPA changes  
Pod count; cluster autoscaling supplies nodes for unschedulable Pods when possible.

A workload may need both HPA and cluster autoscaling: the first requests more replicas,  
and the second supplies capacity when suitable Pods cannot be scheduled. I verify metrics  
availability and realistic requests. Scaling decisions also need startup time, stabilization,  
downstream limits, and application-specific saturation signals to avoid oscillation.

### Q8. Compare startup, readiness, and liveness probes.
**Answer:**

A startup probe protects slow initialization before other probes operate. A  
failed readiness probe removes a Pod from ready Service backends without restarting  
it. Repeated liveness failure restarts the container. Avoid using a transient  
downstream dependency as a liveness check that causes cascading restarts.

For example, readiness can fail while a dependency is temporarily unavailable, allowing  
traffic to avoid that Pod without repeatedly restarting it. A startup probe can accommodate  
initialization that legitimately takes longer. I choose endpoints and thresholds from actual  
behavior and test failure modes to avoid causing an outage through probe configuration.

### Q9. Explain Kubernetes RBAC.
**Answer:**

Roles define permissions in a namespace; ClusterRoles can define cluster-scoped or reusable  
permissions. RoleBindings grant permissions within a namespace, and ClusterRoleBindings  
grant at cluster scope. Bind permissions to appropriate users, groups, or service accounts.  
Validate access with `kubectl auth can-i` and avoid unnecessary cluster-admin grants.

For a deployment identity, I grant only the verbs and resources needed in the  
intended namespace. Reading Secrets or creating privileged workloads can carry more  
risk than the role name suggests. I test the allowed operation and a representative  
forbidden operation so the boundary is demonstrated rather than merely assumed.

### Q10. What does AKS manage, and what remains your responsibility?
**Answer:**

AKS manages the Kubernetes control plane as a service. Teams still need to manage workload  
security, configuration, identity, resource usage, node-pool choices, networking,  
and upgrade planning according to their cluster mode. Understand the shared-responsibility  
boundary rather than assuming a managed cluster operates applications automatically.

I explain which responsibilities belong to the platform and which belong to our team for  
the selected AKS offering. Managed control-plane operation does not validate application  
probes, resource budgets, or backup recovery. Clear ownership matters most during incidents,  
when assumptions about the managed service can otherwise delay the correct response.

### Q11. How do you plan AKS node upgrades?
**Answer:**

Review supported versions, API compatibility, add-ons, available quota, surge capacity,  
storage topology, and disruption budgets. Test representative workloads, upgrade  
in a controlled sequence, and observe drain/replacement behavior. Configure maintenance  
and workload resilience so node replacement does not become an unplanned outage.

I also confirm workloads can tolerate node replacement and that disruption budgets permit  
safe progress. Surge settings may require extra quota and subnet capacity. A successful  
test upgrade should exercise representative storage and networking behavior, with application  
monitoring active, rather than checking only that the cluster reports a new version.

### Q12. What is AKS workload identity?
**Answer:**

It lets an eligible Kubernetes service account federate with a Microsoft Entra identity so the  
application can obtain authorized Azure access without a stored client secret. Configure  
cluster support, federation, Pod/service-account settings, Azure roles, and SDK token usage. See  
[AKS OIDC issuer documentation](https://learn.microsoft.com/en-us/azure/aks/use-oidc-issuer).

For example, an application can use its service account's trusted federation to obtain  
access to Key Vault. I verify the expected identity reaches the SDK and has the  
required vault permissions. Cluster support and federation configuration establish  
authentication, while Azure RBAC and network policy still govern the actual operation.

## Scenario-based questions

### S1. HPA does not add replicas despite high CPU. What do you check?
**Answer:**

Inspect HPA status, events, current metrics, target workload, minimum/maximum replicas, and stabilization  
behavior. Verify resource requests and that `kubectl top pods` or the relevant metrics source works. Check  
whether the observed CPU belongs to the selected metric/container. If replicas increase but stay Pending,  
investigate node capacity separately. Validate scaling under representative load after correcting the constraint.

If current metrics are unknown, I repair the metrics path rather than  
lowering the target blindly. If the HPA is at its maximum, I investigate whether  
a higher maximum is safe for the application and dependencies. After  
the fix, I observe both scale-out and scale-in behavior to check stability.

### S2. A Service has no usable endpoints. What is likely wrong?
**Answer:**

Compare the Service selector with Pod labels in the same namespace and inspect EndpointSlices.  
If matching Pods exist, check readiness; unready Pods are normally not usable backends.  
Verify named target ports and the actual container listener. Fix labels, readiness, or  
port configuration and test a request through the Service, not only directly to a Pod IP.

I also inspect whether the application listens on the Pod network interface and whether  
the Service's protocol matches it. A healthy direct localhost request is insufficient  
if the listener is bound incorrectly. After endpoints appear, I test from an  
appropriate client and confirm that requests reach the expected version and namespace.

### S3. A PVC stays Pending and its Pod cannot start. How do you investigate?
**Answer:**

Inspect PVC and Pod events, StorageClass existence, provisioning errors, requested access modes, quota,  
and CSI health. With delayed binding, provisioning may intentionally wait until scheduling determines  
the topology. Check zone and node constraints together. Correct the storage or scheduling issue  
without deleting a claim containing needed data, then verify mount and application read/write behavior.

I preserve data ownership while investigating because deleting and recreating a claim  
can change the attached storage or trigger deletion behavior. Events often reveal  
quota, topology, or driver errors directly. Recovery is complete only when the  
correct volume is mounted and the application can read expected data and write safely.

### S4. An AKS upgrade is blocked by a PodDisruptionBudget. What should you do?
**Answer:**

Identify the blocked workload, its healthy replicas, and the budget's allowed disruptions. Check  
whether capacity, anti-affinity, or existing failures prevent additional replicas from becoming ready.  
Add suitable capacity or restore replicas first. Change the budget only after assessing availability  
and quorum requirements, then retry the upgrade and monitor application health throughout the drain.

A restrictive budget may be intentional for quorum safety, so I do not weaken it merely  
to make the upgrade green. I assess healthy replicas and placement with the application  
owner. After capacity or availability is corrected, I monitor the drain sequence and  
verify that the cluster reaches the intended version without user-visible degradation.
