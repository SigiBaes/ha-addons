# OpenTelemetry Collector for UniFi

Home Assistant add-on repository with a single add-on: an [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib) set up to receive UniFi Dream Machine SIEM/CEF syslog and forward it over OTLP.

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FSigiBaes%2Fha-addons)

This is a fork of [cedricziel/ha-addons](https://github.com/cedricziel/ha-addons). The upstream repository also ships Grafana Loki, Mimir, and Tempo. This fork keeps only the collector add-on.

## What is different from upstream

- **Host network.** The collector shares the Home Assistant host network, so it can receive UDP syslog on the host and export OTLP to `127.0.0.1` (for example Netdata on the same machine).
- **Built on the device.** There is no `image` key. Supervisor builds the add-on from the Dockerfile. It does not pull `ghcr.io/cedricziel/ha-addon-otelcol-{arch}`.
- **UniFi CEF example.** [`otelcol/examples/unifi-cef-syslog.yaml`](otelcol/examples/unifi-cef-syslog.yaml) is a collector config for UniFi Dream Machine SIEM syslog: UDP `5140`, RFC 3164, lines without a PRI header, CEF headers and extensions parsed into `cef.*` and `cef.ext.*` attributes, CEF severity mapped to INFO/WARN/ERROR/FATAL, and export via OTLP/gRPC.
- **Separate slug.** The add-on slug is `otelcol_unifi`, so it can be installed next to the upstream add-on (`otelcol`).

## Install

1. In Home Assistant, open **Settings → Add-ons → Add-on store → ⋮ → Repositories**.
2. Add `https://github.com/SigiBaes/ha-addons` and refresh the store.
3. Install **OpenTelemetry Collector for UniFi**. The first install builds the image on the device and needs network access to pull the base image and the collector binary.

Then paste the example into the add-on `config` option and point UniFi SIEM syslog at `<HA host>:5140` (UDP). Details, attribute names, and quirks are in [otelcol/DOCS.md](otelcol/DOCS.md).

## Credit

Based on the OpenTelemetry Collector add-on from [cedricziel/ha-addons](https://github.com/cedricziel/ha-addons).

## License

MIT. See [LICENSE](LICENSE).
