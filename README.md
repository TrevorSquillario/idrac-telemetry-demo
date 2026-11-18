# idrac-telemetry-demo
Example code to start monitoring iDRAC Telemetry with Prometheus/Grafana

## Getting Started

This example revolves around the [idrac_exporter](https://github.com/mrlhansen/idrac_exporter) project. It acts as a very simple proxy to provide a `/metrics` endpoint for the iDRAC. It receives a prometheus scrape query like `http://server-name:9348/metrics?target=192.168.1.100`, performs a Redfish query to the `target` and outputs prometheus metrics like [examples/all_metrics.txt](examples/all_metrics.txt)

The compose file expects a locally built `idrac_exporter:latest` image, so build it
from my fork before starting the stack. My fork ads iDRAC Telemetry support. Issue has been opened for this to be added to the main project https://github.com/mrlhansen/idrac_exporter/issues/236 

1. Clone the exporter fork:

   ```sh
   git clone https://github.com/TrevorSquillario/idrac_exporter
   cd idrac_exporter
   ```

2. Build the container image:

   ```sh
   docker build -t idrac_exporter:latest -f Dockerfile .
   ```

3. Clone this repository and configure your iDRAC targets in `config.yaml`:

   ```sh
   git clone https://github.com/TrevorSquillario/idrac-telemetry-demo
   cd idrac-telemetry-demo
   ```

## Configuration

The exporter reads `config.yaml`, which is mounted into the container at
`/etc/prometheus/idrac.yml`. To collect everything the exporter can expose, enable
all metric groups with `metrics.all.enabled: true` and list the telemetry reports
you want to scrape under `telemetry_reports`.

iDRAC credentials can be set in the `config.yaml` or by environment variables passed to the container.
```
hosts:
  default:
    encoded: false        # CONFIG_DEFAULT_ENCODED=false
    username: root        # CONFIG_DEFAULT_USERNAME=user
    password: calvin        # CONFIG_DEFAULT_PASSWORD=pass
```

The available metric report names can also be discovered from the iDRAC itself at
`/redfish/v1/TelemetryService/MetricReportDefinitions`.

You will need to enable each report in each iDRAC. Here is a script to make this easier for you https://github.com/TrevorSquillario/dell-scripts/blob/main/Python/manage_telemetry_reports.py

This example will list all MetricReportDefinition names for a single iDRAC
```
python manage_telemetry_reports.py -ip 192.168.0.120 -u root -p calvin --get-metric-report-definition all
```

This example will enable Telemetry and every MetricReportDefinition for a single iDRAC
```
python manage_telemetry_reports.py -ip 192.168.0.120 -u root -p calvin -s Enabled --metric-report all, 
```

## Running

The compose file uses profiles, so nothing starts unless you pick one:

```sh
# Minimal: idrac_exporter + vmagent + VictoriaMetrics + Grafana
docker compose --profile minimal up -d

# Full: adds vmalert and alertmanager
docker compose --profile full up -d
```

You can also set `COMPOSE_PROFILES=minimal` (or `full`) in your shell or a `.env` file.

| Service | minimal | full |
| --- | --- | --- |
| `idrac_exporter` | ✅ | ✅ |
| `vmagent` | ✅ | ✅ |
| `victoriametrics` | ✅ | ✅ |
| `grafana` | ✅ | ✅ |
| `vmalert` | — | ✅ |
| `alertmanager` | — | ✅ |

Grafana is available at http://localhost:3000, VictoriaMetrics at http://localhost:8428,
and the exporter at http://localhost:9348/metrics?target=192.168.0.202.
