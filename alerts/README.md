# Alert rules (as code)

`prometheus-rules.yml` — generic, vendor-neutral alerting + recording rules implementing the
🔴 Page / 🟠 Ticket rows from [`../CATALOG.md`](../CATALOG.md). **Templates — adapt names and
thresholds to your stack and SLOs.**

**Coverage:** **96 alerts** (48 page + 48 ticket) + 4 recording rules across **25 groups** —
covering essentially every Page/Ticket row in the catalog. The 🟢 *Watch* rows are intentionally
**not** alerts (dashboard/diagnosis only); turning them into alerts is the noise the catalog warns against.

## What's inside

| Domain group(s) | Covers |
|-----------------|--------|
| `slo-recording` + `slo-burn-rate` | Multi-window, multi-burn-rate **error-budget** alerts (the gold standard for paging) |
| `service-latency`, `service-saturation` | p99 latency, connection-pool exhaustion, dependency errors |
| `host-use` | CPU saturation, memory pressure, predictive disk-fill, network errors |
| `kubernetes`, `container-controlplane` | CrashLoop, replicas, NotReady, OOMKill, CPU throttling, etcd fsync/quota, CoreDNS, ingress, pending pods, HPA, API-server 5xx |
| `database-relational`, `database-nosql` | Replication lag, deadlocks, cache-hit, storage, conn-wait; 429 throttling, throughput, hot partition |
| `cache`, `storage` | Hit-ratio, evictions, memory, replication; object-availability, throttling, disk IOPS |
| `serverless` | Error rate, throttles, iterator age, near-timeout |
| `search-engine` | Cluster red, unassigned shards, heap, threadpool rejections |
| `security` | Auth-failure spike, credential stuffing, WAF, DDoS, secret-store throttling, egress anomaly, audit-log gap |
| `cost-finops` | Daily-spend anomaly, budget burn |
| `data-pipeline-quality` | Pipeline failure, freshness, row-count drop, schema drift, validation |
| `ai-ml-llm` | Inference errors, provider 429, GPU, TTFT, drift |
| `backup-dr` | Backup failure, RPO exceeded, DR replication, restore test |
| `multi-tenancy-quota` | Per-tenant SLO burn, noisy neighbor, cloud quota, throttle events |
| `compliance-telemetry` | Residency, encryption coverage, TSDB cardinality, dropped telemetry |
| `edge-and-network` | Gateway upstream errors, rate-limit, LB health/rejects, retransmits, DNS, CDN origin, mesh success/mTLS |
| `realtime-notifications` | Reconnect storms, dropped messages, delivery collapse, complaint rate |
| `messaging` | Consumer lag, dead-letter growth |
| `delivery` | Failed prod deploy, change-failure-rate |
| `certs-and-heartbeats` | Cert expiry, batch dead-man's-switch, scrape target down |

## The burn-rate alerts (read this before tuning)

A static "errors > 1%" pages on harmless blips and misses slow bleeds. Instead we alert on how
fast the **error budget** is being consumed:

- SLO target **99.9%** → error budget = `0.001` (≈ 43 min/month).
- **Fast burn** — 14.4× over **1h + 5m** windows → budget gone in ~2 days → `severity: page`.
- **Slow burn** — 6× over **6h + 30m** windows → `severity: ticket`.

Change the SLO by editing the `0.001` budget factor and (optionally) the burn multipliers.
Two windows (long + short) prevent both flapping and stale alerts. See the
[SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/).

## Severity routing (Alertmanager)

Rules are labelled `severity: page | ticket`. Route them in Alertmanager, e.g.:

```yaml
route:
  receiver: tickets
  routes:
    - matchers: [ 'severity="page"' ]
      receiver: pager
      group_wait: 30s
      repeat_interval: 4h
receivers:
  - name: pager       # PagerDuty / Opsgenie / phone
  - name: tickets     # Jira / Slack channel
```

## Validate & load

```bash
promtool check rules prometheus-rules.yml          # syntax check
promtool test rules <your-tests>.yml               # unit-test alert logic
```

Load it by adding the file to Prometheus `rule_files:` (or a `PrometheusRule` CRD if you run
the Prometheus Operator).

## Managed-cloud equivalents

Where you don't run your own Prometheus, use the provider's recommended-alert baselines instead
(same intent, native metrics):

- **Azure** — Azure Monitor Baseline Alerts (AMBA): <https://azure.github.io/azure-monitor-baseline-alerts/>
- **AWS** — CloudWatch recommended alarms: <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Best_Practice_Recommended_Alarms_AWS_Services.html>
- **GCP** — Cloud Monitoring alerting policies: <https://cloud.google.com/monitoring/alerts>
