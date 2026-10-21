# Day 10: Kubernetes Fundamentals — Interview Questions and Answers

## Interview questions

### Q1. Explain Kubernetes architecture.
**Answer:**

The control plane contains the API server, scheduler, controllers,  
and cluster-state storage in etcd. Nodes run kubelet, a container  
runtime through CRI, and networking components. Users declare desired  
state through the API; controllers reconcile actual state toward it.

For example, submitting a Deployment reaches the API server, a controller creates the  
required workload objects, and the scheduler assigns Pods to nodes. Kubelet then  
starts their containers. This sequence helps locate failures: API acceptance, scheduling,  
image retrieval, startup, and application readiness are separate stages to inspect.

### Q2. What is a Pod?
**Answer:**

A Pod is Kubernetes' smallest deployable workload unit, containing one or more  
colocated containers sharing networking and specified storage. Containers in  
a Pod can communicate through localhost. Pods are replaceable; do not rely on  
a Pod's name, IP, or writable container layer as durable identity or storage.

A sidecar and application container can share a Pod when they require tightly coupled  
lifecycle and local communication. They still have separate processes and container  
filesystems unless volumes are shared. I keep the Pod design focused because adding  
unrelated components makes scaling, failure handling, and resource allocation harder.

### Q3. How do Deployments and ReplicaSets relate?
**Answer:**

A Deployment manages rollout history and desired state for typically  
stateless workloads. It creates and updates ReplicaSets, which maintain the  
requested replica count. Manage replicas and updates through the Deployment  
rather than editing a ReplicaSet that its controller will reconcile.

For example, changing a Deployment's Pod template normally creates a new ReplicaSet  
for the rollout. Old and new ReplicaSets may coexist while availability is  
maintained. I inspect Deployment conditions and revision history to understand  
progress rather than treating every ReplicaSet with fewer replicas as a fault.

### Q4. What does the scheduler do?
**Answer:**

The scheduler chooses a suitable node for an unscheduled Pod based on  
resource requests, constraints, affinity, taints/tolerations, storage requirements,  
and other scheduling policies. It does not start the container itself;  
kubelet on the chosen node handles Pod execution through the runtime.

A Pending Pod may have no suitable node even when several nodes appear online.  
Requests, affinity, storage topology, and taints can eliminate candidates.  
I inspect scheduling events and compare constraints with actual node  
labels and capacity before changing resource requests or scaling the cluster.

### Q5. Why use namespaces?
**Answer:**

Namespaces organize namespaced resources and can scope RBAC, quotas,  
and policies. They help separate teams or environments within a cluster.  
A namespace by itself does not enforce network isolation or provide a  
complete security boundary; use relevant policy and isolation controls.

For a shared cluster, I combine namespaces with scoped permissions, resource  
quotas, and appropriate network policies. Some resources are cluster-scoped and  
cannot be isolated simply by creating another namespace. I evaluate the  
required trust boundary before deciding whether teams can share a cluster safely.

### Q6. What are labels, selectors, and annotations?
**Answer:**

Labels identify resources using key/value metadata and selectors choose matching resources.  
Services and workload controllers depend on correct selectors. Annotations hold  
descriptive or tool-specific metadata that is not intended for label selection. Keep  
identity labels stable to avoid inadvertently redirecting traffic or controller ownership.

For example, a Service selector may target `app=orders`, while an annotation records  
a deployment tool's metadata. If the labels change accidentally, healthy Pods may  
stop receiving Service traffic. I validate selector relationships in rendered manifests  
and avoid using frequently changing values as the core identity of a workload.

### Q7. What are requests and limits?
**Answer:**

Requests describe resource needs used for scheduling and influence resource  
allocation. CPU limits can throttle execution, while memory limits can  
result in an OOM kill when exceeded. Set values from observed workload  
behavior; copying arbitrary limits can create instability or waste capacity.

A Pod can fail scheduling because its requests do not fit even when observed  
utilization looks low. Conversely, a small memory limit can kill a container  
despite free node memory. I use measured load and peak behavior to tune settings  
and verify that the resulting configuration preserves performance and stability.

### Q8. Explain desired-state reconciliation.
**Answer:**

Controllers repeatedly compare declared desired state with observed state and  
act to reduce differences. Deleting a Deployment-managed Pod usually causes  
a replacement. Manual changes to managed children may be overwritten. Update  
the controlling declarative source to make an intentional change durable.

This model explains why replacing a failed Pod is often routine while losing the  
desired-state source is more serious. I inspect owner references to find the controlling  
object. For GitOps-managed systems, the durable source may be a repository rather than  
the live Deployment alone, so operational changes must follow that ownership chain.

### Q9. What does kubectl apply do?
**Answer:**

`kubectl apply -f manifest.yaml` creates or updates declared resources. Behavior  
differs between client-side and server-side apply, including field ownership and  
conflicts. Validate the intended cluster/context and namespace first. A successful  
apply means the API accepted the change, not that the application is healthy.

I review the target context and a meaningful diff before applying sensitive changes.  
After acceptance, I wait for rollout progress and test actual application behavior.  
Server-side field ownership can help identify competing managers, but it does  
not eliminate the need to coordinate changes made by different tools or controllers.

### Q10. Which kubectl commands do you use most often?
**Answer:**

`get` summarizes resources, `describe` exposes details and events, `logs`  
reads container output, and `exec` runs a command in a running container. Use  
`rollout status deployment/app`, `get events`, and `config current-context`  
to inspect deployment progress, recent events, and the selected cluster.

I include the namespace explicitly when investigating production resources and  
select the intended container when a Pod has several. Previous-container logs  
can reveal a crash that current logs miss. These commands work best as a sequence  
driven by the observed failure rather than a list run without interpretation.

### Q11. What is the difference between a Pod phase and a displayed status?
**Answer:**

Pod phases include Pending, Running, Succeeded, Failed, and Unknown. `kubectl  
get pods` may display more specific reasons such as CrashLoopBackOff or  
ImagePullBackOff. Running does not guarantee readiness, and CrashLoopBackOff  
describes repeated restart backoff rather than a standalone Pod phase.

For example, a Pod can be Running while its readiness probe fails, leaving it excluded  
from normal Service backends. I inspect container states and conditions instead  
of relying on one status column. The distinction helps explain why an apparently  
running application can still produce unavailable endpoints and user-facing errors.

### Q12. How do you update and scale an application?
**Answer:**

Change the Deployment's declared image reference to release a version and  
its replica count to scale manually. `kubectl scale deployment/app --replicas=3`  
is useful operationally, but reconcile the source of truth afterward.  
Inspect rollout status, available replicas, and actual request success.

If HPA or GitOps also manages replicas, a manual scale command may be overwritten.  
I identify the authoritative owner before making the change persistent. A release  
is verified through desired versus available replicas, Pod readiness, and real traffic  
because updating an image reference alone says nothing about application health.

## Scenario-based questions

### S1. A Pod remains Pending. How do you troubleshoot it?
**Answer:**

Use `kubectl describe pod POD -n NS` and inspect scheduling events. Look for insufficient requested  
CPU/memory, node selectors, affinity, untolerated taints, unbound PVCs, or unavailable nodes. Inspect node  
capacity and related storage resources. Fix the constraint or add suitable capacity, then confirm scheduling  
and readiness; reducing requests without understanding actual usage can move the failure into runtime.

The event message often names the exact constraint, so I use it to avoid random  
restarts. Adding a node only helps if it satisfies the Pod's labels, topology,  
and resource needs. After scheduling succeeds, I continue through image pull and  
readiness checks because resolving Pending may expose a separate startup problem.

### S2. Deleting a Pod does not remove the application. Why?
**Answer:**

A controller such as a ReplicaSet, StatefulSet, or DaemonSet owns the Pod and recreates  
it to satisfy desired state. Inspect owner references and change the top-level controller  
or its Git-managed declaration. Scale down or remove the correct resource according to the  
intended lifecycle. Confirm another automation controller is not restoring it afterward.

I distinguish an intentional restart from permanent removal. For a restart, replacing  
a controller-managed Pod may be acceptable under the workload's availability policy.  
For removal, I change the controller and source of truth. I also check durable storage  
retention so removing compute does not unintentionally delete or strand required data.

### S3. A Deployment has three replicas but only one is ready. What do you inspect?
**Answer:**

Compare all Pod states, events, readiness failures, logs, and node placement. Inspect image  
pulls, configuration, capacity, dependency access, and startup timing. Check Deployment  
conditions and rollout settings to see whether it is stalled. Correct the underlying cause  
and verify all desired replicas become available and the Service routes successful requests.

I compare the failing Pods with the ready Pod to identify differences in node placement,  
configuration, volume attachment, and dependency access. A rollout can be blocked  
by one constraint while old replicas continue serving. The corrective action should  
preserve available capacity and confirm the intended new version becomes healthy.

### S4. You applied a manifest to the wrong cluster. What should you do?
**Answer:**

Stop additional changes and identify the exact context, namespace, and resources affected. Compare  
against the prior declared state and assess whether any data or traffic changed. Restore the intended  
state through the relevant owner and deployment process, then verify health. Introduce explicit  
context checks and environment-specific credentials so a similar mistake is caught before mutation.

I avoid immediately deleting everything I applied because some resources may already exist  
or serve other workloads. Recovery should be based on the exact diff and ownership. Prevention  
can include explicit context arguments, visible environment prompts, restricted credentials,  
and pipeline checks that reject unexpected cluster identifiers before any write operation.

Troubleshooting reference: [Kubernetes Pod debugging](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/).
