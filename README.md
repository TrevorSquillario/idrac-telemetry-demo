# idrac-telemetry-demo
Example code to start monitoring iDRAC Telemetry with Prometheus/Grafana

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

