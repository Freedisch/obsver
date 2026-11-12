# Klex Insights — starter monitoring stack

Local-only OpenTelemetry Collector → Prometheus → Grafana stack for a **locally running** Klex process.

## Quick start

1. Stop the previous standalone Collector Docker container (its ports 4318 and 8889 conflict).
2. From this directory, run `docker compose up -d`.
3. Start Klex locally with the previously tested flags:
   `klex --telemetry-endpoint http://127.0.0.1:4318 --telemetry-level advanced`
4. Run a few Klex model calls and tool calls.
5. Visit http://localhost:9090/targets — `klex-otel-collector` should be UP.
6. Visit http://localhost:3000 (admin/admin, change on first login). Find **Klex Insights — Agent Operations** under **Dashboards**.
7. Validate the exported metric names with `curl -s http://localhost:8889/metrics | grep 'gen_ai_client_operation_duration_seconds_count'`.

## Important notes

- No custom collector code is needed for basic Klex metrics.
- These first panels rely on metric names in the provided Prometheus output. They show inference **operations**, not user messages.
- Klex telemetry may be cumulative; Grafana panels use `rate` or `increase` to chart changes over a lookback window. New series need multiple observations before some panels show useful data.
- The OpenTelemetry debug exporter prints **basic** diagnostics; remove it if logs/traces might contain sensitive data. `advanced` telemetry is preferable to `debug` during demonstration.
- This is a **local development stack**, with ports bound to localhost and a deliberately simple Grafana password. Do not expose it publicly without authentication, TLS and secret management.
- The collector currently exports traces and logs only to its debug output; no Tempo/Loki persistence is configured yet.
- Provider label `unknown` reflects the original source data; avoid attributing it to a particular model until verified.

## Shutdown

`docker compose down` preserves Prometheus and Grafana data volumes. To erase volumes: `docker compose down -v`.
