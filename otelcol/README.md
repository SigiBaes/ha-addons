# Home Assistant Add-on: OpenTelemetry collector

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg

This add-on runs [OpenTelemetry Collector Contrib](https://github.com/open-telemetry/opentelemetry-collector-contrib) on Home Assistant OS. This fork has no prebuilt image: Supervisor builds it on the device from the Dockerfile. See [DOCS.md](DOCS.md) for how to add `https://github.com/SigiBaes/ha-addons` and install it.

It uses **host networking** so receivers can bind on the host (for example UDP syslog) and exporters can reach OTLP backends on `127.0.0.1`. Paste collector YAML into the `config` option. See [DOCS.md](DOCS.md) for details.

A ready-to-paste example for UniFi Dream Machine SIEM/CEF syslog is in [`examples/unifi-cef-syslog.yaml`](examples/unifi-cef-syslog.yaml): listen on UDP 5140, parse CEF into `cef.*` / `cef.ext.*` attributes, and export OTLP/gRPC to `127.0.0.1:4317`. Point UniFi syslog at `<HA host>:5140`.
