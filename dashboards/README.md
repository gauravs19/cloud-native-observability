# Grafana dashboards (as code)

Three generic, importable Grafana dashboards built on the methods in [`../CATALOG.md`](../CATALOG.md):

| File | Method | Shows |
|------|--------|-------|
| `grafana-golden-signals.json` | GOLD | Traffic · Errors · Latency (p50/p95/p99) · Saturation — service overview |
| `grafana-red-by-endpoint.json` | RED | Rate / Errors / Duration sliced by route, plus per-dependency error ratio |
| `grafana-use-resources.json` | USE | CPU util + saturation, memory, disk used + I/O, network errors — per node |

## Importing

Grafana → **Dashboards → New → Import → Upload JSON**, then pick your Prometheus data
source when prompted (the `DS_PROMETHEUS` input). Or provision them as files via the Grafana
dashboard provisioning config.

## Metric-name assumptions (adapt to your stack)

These use OpenTelemetry / Prometheus conventions. If your names differ, edit the panel
queries — the structure stays the same.

| Used here | Source | Swap for |
|-----------|--------|----------|
| `http_requests_total{service,http_route,status_code}` | app / OTel HTTP server | your request counter |
| `http_request_duration_seconds_bucket` | app / OTel histogram | your latency histogram |
| `http_inflight_requests` | app | worker-pool usage or queue depth |
| `dependency_calls_total{peer_service,outcome}` | app / OTel client | your downstream-call counter |
| `node_*` | node_exporter | cAdvisor / cloud-agent equivalents |

> The dashboards include a **deploy annotation** (`process_start_time_seconds` change) so you
> can see restarts/deploys lined up against the graphs — the cheapest root-cause aid there is.
