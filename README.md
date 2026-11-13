# Klex Insights

A local observability stack for a locally running Klex process. Klex sends
OpenTelemetry data to the Collector, which routes each signal to a dedicated
backend:

```text
Klex ──OTLP/HTTP──> OpenTelemetry Collector
                       ├── metrics ──> Prometheus ──> Grafana
                       ├── traces  ──> Tempo
                       └── logs    ──> Loki
```

The stack includes a pre-provisioned **Klex Insights — Agent Operations**
dashboard for inference metrics.

## Prerequisites

- Docker with Docker Compose
- A local Klex installation that supports OpenTelemetry export

## Quick start

1. Start the observability stack:

   ```sh
   docker compose up -d
   ```

2. Start Klex with OTLP/HTTP telemetry enabled:

   ```sh
   klex --telemetry-endpoint http://127.0.0.1:4318 --telemetry-level advanced
   ```

3. Run a few Klex model and tool calls to generate telemetry.

4. Open the local services:

   | Service | URL | Purpose |
   | --- | --- | --- |
   | Grafana | <http://localhost:3000> | Dashboards (`admin` / `admin` on first login) |
   | Prometheus | <http://localhost:9090> | Metrics and scrape targets |
   | Loki | <http://localhost:3100> | Log storage and query API |
   | Tempo | <http://localhost:3200> | Trace storage and query API |

   In Grafana, open **Dashboards → Klex Insights → Klex Insights — Agent
   Operations**.

## Verify the stack

Check that all containers are running:

```sh
docker compose ps
```

Prometheus should report the `klex-otel-collector` target as `UP` at
<http://localhost:9090/targets>. To confirm that Klex metrics reached the
Collector, inspect its Prometheus endpoint:

```sh
curl -s http://localhost:8889/metrics \
  | grep 'gen_ai_client_operation_duration_seconds_count'
```

You can also check the storage backends directly:

```sh
curl -fsS http://localhost:3100/ready   # Loki
curl -fsS http://localhost:3200/ready   # Tempo
```

## Components

- **OpenTelemetry Collector** receives OTLP/HTTP on port `4318`, exposes
  Prometheus metrics on port `8889`, and forwards logs and traces to Loki and
  Tempo. Its basic debug exporter is also enabled for diagnostics.
- **Prometheus** scrapes Collector metrics every 10 seconds and retains them
  for 7 days.
- **Grafana** provisions Prometheus as its default data source and loads the
  bundled Klex dashboard. Loki and Tempo store incoming data but are not yet
  provisioned as Grafana data sources.
- **Loki** stores logs in the `loki-data` Docker volume.
- **Tempo** stores traces in the `tempo-data` Docker volume.

All published ports bind to `127.0.0.1` only.

## Notes

- Dashboard panels represent inference **operations**, not user messages.
- Some panels use `rate` or `increase` because Klex telemetry may be
  cumulative. New time series need multiple observations before these panels
  display useful data.
- A provider label of `unknown` comes from the source telemetry and should not
  be attributed to a model provider without verification.
- The Collector's debug exporter writes basic diagnostics to container logs.
  Review or disable it if telemetry may contain sensitive information.
- This configuration is intended for local development. The default Grafana
  credentials, disabled Loki authentication, and lack of TLS are not suitable
  for public deployment.

## Troubleshooting

If startup fails because ports `4318` or `8889` are already in use, stop any
standalone OpenTelemetry Collector that is using them, then rerun
`docker compose up -d`.

Inspect service logs with:

```sh
docker compose logs -f collector
docker compose logs -f prometheus
docker compose logs -f loki
docker compose logs -f tempo
```

## Shutdown

Stop the stack while preserving Prometheus, Grafana, Loki, and Tempo data:

```sh
docker compose down
```

To also delete all persisted data:

```sh
docker compose down -v
```
