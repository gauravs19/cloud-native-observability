# Multi-Cloud Service & Metric Map

The same observability *roles* implemented across **Azure**, **AWS**, and **GCP**. Pick the row
for the capability you run; the **What to watch** column is vendor-neutral (the signal that
matters regardless of product), with provider metric names called out where they differ.

> Provider metric catalogs: **Azure** → Azure Monitor `metrics-index`; **AWS** → CloudWatch
> per-namespace metrics; **GCP** → Cloud Monitoring `metric-list`. Links at the bottom.

Legend (carried from `CATALOG.md`): 🔴 Page · 🟠 Ticket · 🟢 Watch.

---

## Compute

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| VM / IaaS | Virtual Machines | EC2 | Compute Engine | 🟠 CPU util + saturation, 🔴 memory available, 🟠 disk IOPS-consumed %, network in/out |
| Autoscaling group | VM Scale Sets | Auto Scaling Groups | Managed Instance Groups | 🟠 desired vs in-service instances, scaling-activity failures, pinned-at-max |
| PaaS web app | App Service | Elastic Beanstalk / App Runner | App Engine / Cloud Run | 🔴 5xx rate (SLO burn), 🟠 CPU/mem %, 🟠 request-queue length, p95 latency |
| Serverless functions | Functions | Lambda | Cloud Functions | 🔴 error rate, 🔴 throttles/concurrency limit, 🟠 duration vs timeout, cold-start rate, DLQ |
| Containers (managed K8s) | AKS | EKS | GKE | 🔴 pod ready<desired / CrashLoop, 🔴 node NotReady, 🟠 CPU/mem working-set, control-plane/etcd health |
| Containers (serverless) | Container Apps | ECS Fargate / App Runner | Cloud Run | 🔴 5xx, 🟠 scale ceiling, replica restarts |
| Batch | Batch | AWS Batch | Cloud Batch / Dataflow | 🔴 task/job failures, 🟠 queue wait, last-success age |

## Networking

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| L4 load balancer | Load Balancer | NLB | Network/Passthrough LB | 🔴 healthy-host count, 🔴 SNAT/port exhaustion, byte/SYN counts |
| L7 load balancer | Application Gateway | ALB | HTTP(S) LB | 🔴 unhealthy hosts, 🔴 backend 5xx, target/backend latency, surge queue |
| CDN / edge | Front Door / Azure CDN | CloudFront | Cloud CDN / Media CDN | 🟠 cache-hit ratio, 🔴 origin 5xx, origin latency, egress bytes |
| Global DNS routing | Traffic Manager | Route 53 | Cloud DNS + GCLB | 🔴 endpoint/health-check status, query volume |
| Site-to-site VPN | VPN Gateway | Site-to-Site VPN | Cloud VPN | 🔴 tunnel state, 🟠 tunnel bandwidth, ingress/egress packets |
| Dedicated interconnect | ExpressRoute | Direct Connect | Cloud Interconnect | 🔴 BGP/peering availability, bits in/out, ARP/light levels |
| Cloud firewall | Azure Firewall | Network Firewall | Cloud NGFW | 🔴 SNAT exhaustion, throughput, rule hits, health |
| DDoS protection | DDoS Protection | Shield | Cloud Armor | 🔴 under-attack flag, dropped packets/bytes |
| Outbound NAT | NAT Gateway | NAT Gateway | Cloud NAT | 🔴 SNAT port exhaustion, dropped packets, allocation errors |
| Private connectivity | Private Link / Endpoint | PrivateLink (VPC Endpoint) | Private Service Connect | 🟢 bytes processed, connection state |

## Data — Relational

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Managed relational | SQL Database | RDS / Aurora | Cloud SQL / AlloyDB | 🟠 CPU/DTU/ACU %, 🔴 storage %, 🟠 connections vs max, 🔴 replica lag, deadlocks, buffer-cache hit |
| Cloud-native distributed SQL | — (Hyperscale) | Aurora | Cloud Spanner | 🟠 CPU %, 🔴 storage, commit/lock latency, replica/leader lag |

## Data — NoSQL / Cache

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Document / wide-column | Cosmos DB | DynamoDB | Firestore / Bigtable | 🔴 throttled (429) requests, 🟠 consumed vs provisioned RU/RCU/WCU, 🟠 hot-partition skew, P99 latency |
| Managed cache | Cache for Redis | ElastiCache | Memorystore | 🟠 hit ratio, 🟠 evictions, 🟠 memory %, engine/server load, command latency |

## Storage

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Object store | Blob Storage | S3 | Cloud Storage | 🔴 availability, 🟠 throttled requests (hot prefix), E2E latency, 4xx/5xx, egress |
| Block disk | Managed Disks | EBS | Persistent Disk / Hyperdisk | 🟠 IOPS/throughput consumed % (tier cap), queue depth, await |
| Managed file share | Azure Files / NetApp | EFS / FSx | Filestore | 🟠 capacity, throughput/burst credits, latency |

## Messaging & Integration

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Queue | Service Bus / Storage Queues | SQS | Pub/Sub (pull) | 🔴 backlog growth, 🟠 oldest-message age, DLQ count, throttling |
| Event streaming | Event Hubs | Kinesis / MSK | Pub/Sub / Managed Kafka | 🔴 consumer lag (egress vs ingress / iterator age), under-replicated/offline partitions, throttling |
| Event routing | Event Grid | EventBridge | Eventarc | 🟠 delivery failures, DLQ count, delivery latency |
| API gateway | API Management | API Gateway | API Gateway / Apigee | 🟠 capacity/utilization, 🔴 5xx, 4xx/unauthorized, backend latency |
| Workflow / orchestration | Logic Apps | Step Functions | Workflows | 🔴 run/execution failures, run latency, throttled executions |

## Analytics / Streaming / Data

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Stream processing | Stream Analytics | Kinesis Data Analytics / Managed Flink | Dataflow | 🔴 watermark/event-time lag, input backlog, 🟠 SU/worker utilization, runtime errors |
| Batch ETL / pipelines | Data Factory | Glue / Step Functions | Dataflow / Composer | 🔴 failed runs, run duration, integration-runtime/worker saturation |
| Data warehouse | Synapse (Dedicated Pool) | Redshift | BigQuery | 🟠 slot/DWU utilization, queued queries, spill, cache hit (BigQuery: slot ms, bytes scanned) |
| Big-data / Spark | Databricks / HDInsight | EMR | Dataproc | 🔴 job failures, executor CPU/mem, shuffle spill, queue |
| Real-time analytics DB | Data Explorer (ADX) | — (OpenSearch) | — | 🟠 ingestion latency/failures, query duration, cache utilization |

## AI / ML

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Managed LLM API | Azure OpenAI | Bedrock | Vertex AI (Gemini) | 🟠 token usage, 🟠 429 throttling, latency/TTFT, provisioned-capacity (PTU) utilization |
| ML model serving | Azure ML Endpoints | SageMaker Endpoints | Vertex AI Endpoints | 🔴 endpoint error rate, latency, 🟠 GPU/CPU utilization, deployment health |
| Cognitive / AI services | AI Services | Rekognition/Comprehend/etc. | Cloud Vision/NLP/etc. | 🟠 4xx/429, call volume, latency |

## Identity & Security

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| Identity provider | Entra ID | IAM / Cognito | Cloud Identity / IAM | 🔴 sign-in failure spike, risky sign-ins, conditional-access/MFA failures |
| Secrets / key vault | Key Vault | Secrets Manager / KMS | Secret Manager / Cloud KMS | 🟠 API 4xx/5xx, 🟠 429 throttling, latency |
| Posture / CSPM | Defender for Cloud | Security Hub / GuardDuty | Security Command Center | 🟠 new high/critical findings, secure-score drop |
| SIEM | Microsoft Sentinel | (OpenSearch / partners) | Chronicle | 🔴 ingestion gap (blind spot), incident count, rule health |
| WAF | App Gateway / Front Door WAF | AWS WAF | Cloud Armor | 🟠 blocked requests, rule matches by category |

## Platform / Ops (the observability plane)

| Role | Azure | AWS | GCP | What to watch |
|------|-------|-----|-----|---------------|
| APM / tracing | Application Insights | X-Ray + CloudWatch | Cloud Trace + Monitoring | 🔴 request failure-rate & response-time SLO burn, dependency RED, exceptions |
| Logs | Log Analytics | CloudWatch Logs | Cloud Logging | 🟠 ingestion volume/latency, daily-cap/quota hit, dropped logs |
| Metrics / alerting | Azure Monitor + AMBA | CloudWatch Alarms | Cloud Monitoring + Alerting | 🔴 alert-pipeline health, action/notification delivery failures |
| Platform status | Resource / Service Health | Health Dashboard / Health API | Service Health / Status | 🔴 provider-side outage affecting you |
| Cost | Cost Management | Cost Explorer / Budgets | Billing / Budgets | 🔴 daily-spend anomaly, 🟠 budget burn |

---

## Provider metric catalogs (master lists)

- **Azure Monitor — supported metrics index:** <https://learn.microsoft.com/azure/azure-monitor/reference/supported-metrics/metrics-index>
- **AWS CloudWatch — metrics & dimensions by service:** <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CW_Support_For_AWS.html>
- **GCP Cloud Monitoring — metrics list:** <https://cloud.google.com/monitoring/api/metrics_gcp>

## Recommended-alert baselines per provider

- **Azure:** Azure Monitor Baseline Alerts (AMBA) — <https://azure.github.io/azure-monitor-baseline-alerts/>
- **AWS:** CloudWatch recommended alarms — <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Best_Practice_Recommended_Alarms_AWS_Services.html>
- **GCP:** Cloud Monitoring alerting + recommended policies — <https://cloud.google.com/monitoring/alerts>

---

*Vendor product names change; the **What to watch** column is the durable part. Map it to your provider's current metric names from the catalogs above.*
