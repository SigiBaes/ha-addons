# OpenTelemetry Collector for UniFi

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports i386 Architecture][i386-shield]

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg

Home Assistant add-on that runs [OpenTelemetry Collector Contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib) for UniFi Dream Machine SIEM/CEF syslog.

This is a fork of the collector add-on in [cedricziel/ha-addons](https://github.com/cedricziel/ha-addons). Compared with upstream:

- It uses the host network, so it can receive UDP syslog on the Home Assistant host and send OTLP to `127.0.0.1` (for example Netdata).
- Supervisor builds it on the device. It does not pull the upstream image.
- [`examples/unifi-cef-syslog.yaml`](examples/unifi-cef-syslog.yaml) is ready to paste into the `config` option. It listens on UDP 5140, parses CEF headers into `cef.*` and extensions into `cef.ext.*`, maps CEF severity, and exports OTLP/gRPC to `127.0.0.1:4317`.
- The slug is `otelcol_unifi`, so it can be installed beside the upstream `otelcol` add-on.

Point UniFi SIEM syslog at `<HA host>:5140`. Install steps, the attribute scheme, and known quirks are in [DOCS.md](DOCS.md).
