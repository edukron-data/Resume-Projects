# Day 4: Azure Fundamentals and Networking — Interview Questions and Answers

## Interview questions

### Q1. Explain subscriptions, resource groups, and regions.
**Answer:**

A subscription is a billing, quota, and access-management boundary. A resource group organizes  
related resources for lifecycle management. A region is a deployment geography containing  
datacenters. A resource group's metadata location does not require all contained  
resources to use that region. Choose resource boundaries according to ownership and lifecycle.

For example, a production subscription can separate billing and access from development,  
while resource groups divide applications by lifecycle. I avoid grouping unrelated  
systems merely because they share a region. Deleting a resource group can remove  
its contained resources, so ownership and lifecycle boundaries matter operationally.

### Q2. What is a VNet, and why use subnets?
**Answer:**

A virtual network provides private IP networking for Azure resources. Subnets divide  
its address space for resource placement, delegation, routing, and security  
controls. Plan nonoverlapping ranges for peering and hybrid connectivity. A subnet  
alone does not create an application security boundary without appropriate controls.

I reserve address ranges for expected growth and document connectivity to on-premises  
networks or other clouds. Application, data, and management components may need  
different subnet controls. The address plan should account for platform-specific subnet  
requirements and future peering so expansion does not require disruptive renumbering.

### Q3. How do network security groups work?
**Answer:**

NSGs filter inbound and outbound layer-3/4 traffic using prioritized rules.  
Lower numerical priority is evaluated first. Rules can reference addresses,  
ports, protocols, service tags, and application security groups. NSGs  
are stateful; both subnet and NIC rules must permit a flow when both apply.

For example, an application subnet may allow traffic only from its approved  
ingress path to the application port. I inspect effective rules when both NIC  
and subnet NSGs apply. Broad allow rules can defeat narrower intentions, so  
I review priorities and actual source/destination values before deployment.

### Q4. What is a route table?
**Answer:**

A route table contains user-defined routes associated with subnets, often directing  
traffic through a firewall or network appliance. Azure combines these with system routes  
and propagated routes. Longest-prefix matching matters; for equal prefixes, route-source  
preference matters. Inspect effective routes instead of guessing the actual path.

A route can send traffic to a firewall successfully while the return path bypasses  
it, causing asymmetric connectivity problems. I inspect both directions and  
any appliance forwarding requirements. A network change is complete only after  
the intended application flow works and unauthorized bypass paths remain blocked.

### Q5. What are VNet peering's limitations?
**Answer:**

Peering connects VNets over Microsoft's backbone and requires nonoverlapping  
address spaces. It is not automatically transitive: peering A to B and B  
to C does not create A-to-C connectivity. Gateway transit, forwarded traffic,  
routes, and network policies must be configured for the intended topology.

In a hub-and-spoke design, I explicitly decide how spokes reach shared  
services and each other. Peering does not automatically configure application  
listeners, DNS, or host firewalls. I validate each required flow and  
document whether it uses direct peering, a firewall, or gateway transit.

### Q6. How does a private endpoint differ from a service endpoint?
**Answer:**

A private endpoint gives a supported service a private IP in a VNet through  
Private Link. A service endpoint extends subnet identity to a supported  
service's public endpoint. Private endpoint access requires correct routing and  
DNS; creating one does not universally disable the service's public access.

For example, a storage private endpoint lets private clients resolve and reach the service  
through its private address, but their identity still needs authorization. I verify the service's  
public-network configuration separately. Security reviews should cover name resolution,  
network paths, and identity together rather than treating private addressing as sufficient.

### Q7. Why is DNS essential for private endpoints?
**Answer:**

Applications normally connect using the service hostname, which must resolve to  
the intended private endpoint address from private clients. Configure the appropriate  
private DNS zone, records, VNet links, and conditional forwarding for hybrid  
networks. Replacing hostnames with IPs may break TLS validation and service routing.

I test resolution from the same network context as the failing application,  
including on-premises or connected spokes. A record visible in one VNet may  
not be visible elsewhere. Correct zone links and forwarding preserve the service  
hostname while making its private address available to intended clients.

### Q8. Compare Azure Load Balancer and Application Gateway.
**Answer:**

Azure Load Balancer distributes layer-4 TCP/UDP flows. Application Gateway provides  
layer-7 HTTP(S) routing, TLS termination, host/path-based routing, and optional  
WAF capabilities. Choose according to protocol, routing requirements, security  
features, and operational needs rather than assuming every application needs both.

For a TCP service without HTTP routing requirements, a layer-four load balancer  
may be appropriate. For host-based web routing or WAF inspection, Application  
Gateway may be a better fit. I also consider health probes, TLS ownership,  
availability, cost, and the operational team responsible for each component.

### Q9. What is the difference between public and private IP addresses?
**Answer:**

A private IP provides addressing within connected private networks; a public IP  
supports internet-facing connectivity when associated resources and policies  
allow it. An IP's existence does not prove a service is listening or reachable.  
Check frontend configuration, routing, security rules, and application bindings.

A private endpoint or VM can still be unreachable from a private client if  
the networks are not connected correctly. Similarly, a public frontend may  
intentionally restrict source access. I validate the full path and avoid assuming  
that an address classification alone determines accessibility or security.

### Q10. What is a hub-and-spoke network?
**Answer:**

The hub centralizes shared connectivity, DNS, firewalling, or hybrid gateways.  
Spokes host workloads with separate ownership or environments. Peering  
and route design determine spoke-to-spoke traffic. Prevent unintended bypass  
paths and asymmetric routing, and size shared services to avoid bottlenecks.

Centralization reduces duplicated shared services but also creates dependencies that  
need capacity, resilience, and clear ownership. I document address spaces, peering  
relationships, DNS forwarding, and routing policies. When troubleshooting, the diagram  
helps identify whether the fault belongs to a spoke workload or a shared hub component.

### Q11. How do availability zones differ from regions?
**Answer:**

Availability zones are physically separated locations within a region designed for  
independent infrastructure failures. Regions are separate geographic deployment areas.  
Zone resilience depends on the service and deployment configuration; cross-region  
recovery also needs data replication, traffic routing, and tested recovery procedures.

For example, distributing instances across zones can protect against one zone failure,  
while regional recovery needs resources and data in another region. I check the  
specific service's supported configuration. Recovery design must include dependencies  
because zone-resilient compute cannot compensate for a single unavailable database.

### Q12. How do you troubleshoot a network connection systematically?
**Answer:**

Follow the path: client DNS, client route, network security rules, intermediate  
firewall, destination listener, TLS, and application response. Use `nslookup`  
or `dig`, `curl`, effective routes, effective NSGs, and Azure Network Watcher  
capabilities. Record source, destination, port, timestamp, and exact error.

I change one likely cause at a time and record the observed result. A timeout,  
connection refusal, TLS error, and application rejection represent different failure  
stages. Using those distinctions narrows the investigation and avoids weakening  
network security to solve what is actually an authentication or configuration issue.

## Scenario-based questions

### S1. A VM cannot connect to Azure SQL after a private endpoint is created. What do you check?
**Answer:**

Resolve the SQL hostname from the VM and verify the intended private IP. Check private endpoint approval,  
private DNS records and links, custom DNS forwarding, routing, and relevant security rules. Test the  
required TCP connectivity, then authentication separately. Inspect public-network settings and service  
configuration; a private endpoint is not an authentication grant. Verify a real query from the application path.

If DNS returns a public address unexpectedly, I fix zone visibility or forwarding  
before changing database permissions. If the private address resolves  
correctly but TCP fails, I focus on the network path. This separation prevents  
unrelated changes and makes the final diagnosis explainable to the interviewer.

### S2. Two peered VNets cannot communicate. What is your approach?
**Answer:**

Confirm both peering directions are connected, address ranges do not overlap, and the destination IP  
is correct. Inspect effective routes, NSGs, host firewalls, and the application's listener. If traffic  
crosses an appliance, verify forwarding and return routes. Test the actual application port rather than  
relying exclusively on ICMP. Fix the specific blocked hop and retest both directions where required.

I compare actual source and destination address ranges against peering and firewall  
rules because translated or appliance-forwarded traffic may use different  
identities than expected. The return path is equally important. After the repair, I  
test the required service and confirm that unintended networks did not gain access.

### S3. Application Gateway returns 502 after a deployment. What would you investigate?
**Answer:**

Inspect backend health and the deployment timeline. Verify backend port, protocol, host header,  
probe path, accepted probe status, certificate trust, and network reachability. Compare a direct  
backend request with the gateway request. Correct mismatched settings or restore the healthy  
application version; confirm backend probes and external requests succeed before resuming rollout.

The gateway's backend-health reason helps distinguish probe configuration problems  
from application failures. For HTTPS backends, hostname and certificate mismatches can  
fail even when a raw port test succeeds. I preserve the required TLS validation and  
correct the expected host, trust, or listener settings instead of disabling protection.

### S4. How would you expose a web application while keeping its database private?
**Answer:**

Expose an appropriate HTTP ingress such as Application Gateway with TLS and WAF when required. Place  
application compute on private networking, restrict inbound access to the expected path, and use a  
database private endpoint with private DNS. Apply least-privilege identity and database permissions.  
Validate internet-to-app connectivity and confirm database access is limited to approved private clients.

I also define operational access, backup paths, and monitoring so the private database  
remains manageable during incidents. Private networking can complicate deployment agents  
and administrative tools. A good design demonstrates both that approved workloads can  
connect and that internet clients cannot bypass the intended application access path.

Private endpoint policy detail: subnet network policies must be configured for applicable NSG/route controls. See [Azure private endpoint network policies](https://learn.microsoft.com/en-us/azure/private-link/disable-private-endpoint-network-policy).
