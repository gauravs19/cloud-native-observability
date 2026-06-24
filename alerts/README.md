# Alert rules (as code)

`prometheus-rules.yml` — generic, vendor-neutral alerting + recording rules implementing the
🔴 Page / 🟠 Ticket rows from [`../CATALOG.md`](../CATALOG.md). **Templates — adapt names and
thresholds to your stack and SLOs.**

## What's inside

| Group | Covers |
|-------|--------|
| `slo-recording` + `slo-burn-rate` | Multi-window, multi-burn-rate **error-budget** alerts (the gold standard for paging) |
| `service-latency` | p99 latency vs SLO |
| `host-use` | CPU saturation, memory pressure, predictive disk-fill, network errors |
| `kubernetes` | CrashLoop, replicas unavailable, NotReady node, OOMKill, API-server 5xx |
| `service-saturation` | Connection-pool exhaustion, downstream dependency errors |
| `messaging` | Consumer lag, dead-letter growth |
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
