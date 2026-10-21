# Day 17: Monitoring and Observability — Interview Questions and Answers

## Interview questions

### Q1. How does monitoring differ from observability?
**Answer:** Monitoring evaluates known signals and conditions, while observability helps explain internal behavior from emitted telemetry, including unfamiliar failures. Metrics, logs, traces, and application context work together. Collecting large volumes of data is not enough without useful correlation, access, retention, and questions the data can answer.

### Q2. What does Azure Monitor provide?
**Answer:** Azure Monitor is a platform for collecting and analyzing telemetry from supported Azure and other sources, with metrics, logs, alerts, and integrations. Actual coverage depends on configured collection, diagnostic settings, agents, and service instrumentation. Resource existence does not guarantee all useful logs are collected automatically.

### Q3. What is a Log Analytics workspace?
**Answer:** It is a resource used to store and query Azure Monitor log data under configured access, retention, and billing controls. Tables and schemas depend on the data sources and configuration. Design workspace boundaries around security and operations needs while considering cross-workspace investigation and cost.

### Q4. What is Application Insights?
**Answer:** Application Insights provides application performance monitoring capabilities such as requests, dependencies, exceptions, and distributed tracing when suitably instrumented. Correlation requires propagated context across services. Sampling and collection configuration affect what can be investigated, so missing telemetry does not prove a request never occurred.

### Q5. What is KQL used for?
**Answer:** Kusto Query Language filters, transforms, joins, and summarizes supported telemetry. A query such as `AppRequests | where TimeGenerated > ago(30m) | summarize Requests=count() by bin(TimeGenerated, 5m)` counts workspace-based request records over time. Confirm the actual table/schema; weighted sampled counts may require `ItemCount`.

### Q6. Compare metrics, logs, and traces.
**Answer:** Metrics are numerical time series suited to trends and alerts. Logs record detailed events. Traces describe a request's path across components using spans and context. Use metrics to detect an increase in latency, traces to locate the slow dependency, and logs to inspect its specific failure.

### Q7. How does Prometheus work?
**Answer:** Prometheus typically scrapes metrics endpoints and stores labeled time series for PromQL queries and alert rules. Labels enable filtering but excessive cardinality increases cost and resource use. Avoid user IDs or request IDs as unbounded labels. Plan retention, availability, and long-term storage according to requirements.

### Q8. What is Grafana's role?
**Answer:** Grafana visualizes and can alert on data from configured sources such as Prometheus or Azure Monitor. It is not inherently the source of truth for collected metrics. Protect data-source credentials and dashboard access, and make dashboards show environment, time range, units, and useful operational context.

### Q9. Explain SLA, SLI, and SLO.
**Answer:** An SLI measures a service behavior such as successful-request proportion. An SLO sets a target for that indicator over a defined window. An SLA is an agreement that may include consequences for failing commitments. Define user-visible success carefully so a healthy server count does not hide failed transactions.

### Q10. What is an error budget?
**Answer:** An error budget is the amount of unreliability allowed by an SLO. With a 99.9% request-success target over one million eligible requests, the budget is one thousand failed requests. Time-based and request-based budgets differ. Use budget consumption to guide release and reliability decisions.

### Q11. What makes an alert actionable?
**Answer:** It signals meaningful impact or an urgent precursor, has a clear owner, includes context and a runbook, and supports an appropriate response. Set evaluation windows and thresholds to limit noise without hiding incidents. Test notification delivery and avoid paging repeatedly for each symptom of one root failure.

### Q12. Why monitor latency percentiles instead of averages alone?
**Answer:** Averages can hide slow requests affecting a minority of users. Percentiles such as p95 and p99 describe the tail of the observed distribution. Use adequate sample sizes and appropriate histogram aggregation; averaging per-instance percentiles does not generally produce the correct overall service percentile.

## Scenario-based questions

### S1. Customers report slowness while CPU looks normal. What do you inspect?
**Answer:** Check end-to-end request latency by route, region, and deployment version, then follow slow traces into database calls, outbound dependencies, queues, and connection pools. Inspect memory, thread/worker saturation, throttling, DNS, and network latency. Compare with the normal baseline and isolate the affected segment. Validate improvement through the same user-visible latency indicator, not CPU alone.

### S2. The team receives hundreds of alerts for one incident. How would you improve alerting?
**Answer:** Identify shared causes and group or suppress duplicate downstream notifications using supported alert routing. Page on user impact and critical dependencies, while keeping detailed symptoms available for diagnosis. Add ownership and runbooks, tune windows using incident history, and test maintenance suppression. Check that the new design still detects the original incident quickly and does not suppress unrelated failures.

### S3. An application sends no telemetry after deployment. Where do you start?
**Answer:** Verify instrumentation configuration, connection settings, exporter/agent health, ingestion endpoint reachability, and any identity permissions. Confirm the expected workspace, table, time range, and ingestion delay. Check sampling and disabled collection paths. Generate a known test request and trace it from local emission to ingestion. Restore collection without assuming missing data means no errors occurred.

### S4. A service burns its monthly error budget in two days. What should happen?
**Answer:** Confirm the SLI calculation, eligible traffic, and source data. Assess current impact and follow the agreed error-budget policy, which may pause risky releases while prioritizing reliability fixes. Identify dominant failure modes and assign corrective work. Use short- and long-window burn-rate alerts to detect recurrence, and resume normal release pace according to measured recovery and policy.
