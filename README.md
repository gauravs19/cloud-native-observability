# Cloud-Native Observability — Master Metrics & Alerts Reference

[![validate](https://github.com/gauravs19/cloud-native-observability/actions/workflows/validate.yml/badge.svg)](https://github.com/gauravs19/cloud-native-observability/actions/workflows/validate.yml)

A generic, vendor-neutral master catalog of **everything worth observing** in a cloud-native
solution — the signal types, every layer of the stack, the cross-cutting dimensions
(security, cost, business, data quality, AI/LLM, delivery), and a categorized Azure service
map. Metric names follow OpenTelemetry / Prometheus conventions.

📖 **The full catalog lives in [`CATALOG.md`](./CATALOG.md).**

> **All thresholds are starting points — tune against your own SLOs and baselines.**

## Repo contents

| Path | What it is |
|------|------------|
| [`CATALOG.md`](./CATALOG.md) | The master catalog — 40 sections, every metric as `Metric · Tags · description` |
| [`docs/cloud-provider-map.md`](./docs/cloud-provider-map.md) | The same roles mapped across **Azure / AWS / GCP**, with the vendor-neutral signal to watch |
| [`dashboards/`](./dashboards/) | Importable **Grafana** dashboards as code — Golden Signals, RED-by-endpoint, USE-by-resource |
| [`alerts/`](./alerts/) | Generic **Prometheus** alert rules — multi-window burn-rate SLO alerts + USE/RED/K8s/messaging/cert rules |

Use the catalog to decide *what* to measure, the provider map to find the *metric name* on
your cloud, and the dashboards + alerts as a *starting implementation*.

---

## How it's organized

Each metric is one table row: **Metric · Method · Action · Detailed description** (what it
measures, why it matters, and the recommended alert with its rationale).

The **Method** says why the metric exists; the **Action** says what an alert should do:

| Method | Meaning | | Action | Meaning |
|--------|---------|---|--------|---------|
| `RED` | Rate · Errors · Duration | | 🔴 | **Page** — wake a human (user-visible SLO burn / data loss) |
| `USE` | Utilization · Saturation · Errors | | 🟠 | **Ticket** — attention soon, auto-file an issue |
| `GOLD` | Golden Signals | | 🟢 | **Watch** — dashboard/diagnosis only, don't alert |
| `4KM` | Four Key Metrics (DORA) | | | |
| `BIZ` | Business / product signal | | | |

## Contents of `CATALOG.md`

1. **Methods & their theory** — where RED / USE / Golden Signals / DORA come from
2. **Glossary & abbreviations** — ~80 terms expanded
3. **Part A** — the signal types (metrics, logs, traces, RUM, synthetic, …)
4. **Part B** — 18 stack layers (frontend → API gateway → service → network → DB → K8s → …)
5. **Part B′** — 8 protocol/workload layers (gRPC, GraphQL, real-time, search, vector DB, mobile, …)
6. **Part C** — 7 cross-cutting dimensions (security, cost, business, data quality, AI/LLM, DORA, DR)
7. **Part C′** — 7 operational dimensions (alerting health, capacity, multi-tenancy, quotas, compliance, …)
8. **Part D** — Azure service map (~60 services, by category)
9. **Part E** — operating the system (SLI/SLO/SLA, thresholds, severity, alert lifecycle)

## Key references

- **Azure Monitor Baseline Alerts (AMBA)** — <https://azure.github.io/azure-monitor-baseline-alerts/>
- **Google SRE Book / Workbook** — <https://sre.google/books/>
- **OpenTelemetry semantic conventions** — <https://opentelemetry.io/docs/specs/semconv/>
- **RED method** · **USE method** · **DORA / Four Keys** · **CNCF Observability Whitepaper**

(Full link list at the end of `CATALOG.md`.)

## License

Released under the [MIT License](./LICENSE) — free to use, adapt, and share with attribution.

---

*Generic, vendor-neutral reference. Tune all thresholds to your SLOs.*
