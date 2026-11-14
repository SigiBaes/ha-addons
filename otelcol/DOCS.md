# OpenTelemetry Collector for UniFi

This add-on runs the [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib) on Home Assistant so a UniFi Dream Machine can send SIEM/CEF syslog to it. The example config parses CEF headers into `cef.*` and extensions into `cef.ext.*`, maps severity, and forwards the logs with OTLP (for example to Netdata on the same host).

It is a fork of the collector add-on in [cedricziel/ha-addons](https://github.com/cedricziel/ha-addons). Upstream publishes a prebuilt image and uses the slug `otelcol`. This fork:

- runs on the host network
- is built on the device, with no upstream image
- ships [`examples/unifi-cef-syslog.yaml`](examples/unifi-cef-syslog.yaml)
- uses the slug `otelcol_unifi`, so both add-ons can be installed together

## Install

1. In Home Assistant, open **Settings → Add-ons → Add-on store → ⋮ → Repositories**.
2. Add `https://github.com/SigiBaes/ha-addons` and wait for the store to refresh.
3. Install **OpenTelemetry Collector for UniFi**. The first start waits while Supervisor builds the image. The device needs network access to pull the Debian base image and the collector binary.

The add-on page links back to `https://github.com/SigiBaes/ha-addons`.

## How to use

The default `config` option only receives OTLP and writes it to the debug exporter, including the collector's own metrics. Replace that option with the UniFi example below to receive CEF syslog.

## Host networking

The add-on sets `host_network: true`. The collector uses the Home Assistant host's network stack, so it can:

- Receive UDP syslog on a host port (the UniFi example below uses 5140)
- Export to an OTLP backend that listens on `127.0.0.1` on the same host (for example Netdata)

Supervisor does not apply the `ports` map when host networking is on. Docker host networking has no separate published ports; the collector binds the addresses in its own config directly on the host. The `4317/tcp` and `4318/tcp` entries are left as `null` so the add-on page still names the default OTLP listeners. They are not host-port mappings. The default collector config listens on those ports on the host. If another process already uses 4317 or 4318, replace that config before starting the add-on. UDP 5140 is not listed in `ports`: only the optional example listens on it, and that bind comes from the collector config.

## Example: UniFi Dream Machine CEF syslog

[`examples/unifi-cef-syslog.yaml`](examples/unifi-cef-syslog.yaml) is a complete collector config (validated against otelcol-contrib 0.163.x) that receives UniFi SIEM/CEF syslog over UDP and forwards parsed logs to an OTLP/gRPC backend on the same host.

What it does:

- Listens for syslog on UDP `0.0.0.0:5140` using RFC 3164
- Sets `allow_skip_pri_header: true` because UniFi often sends lines without a PRI header
- Parses CEF payloads in `transform/cef` into attributes (see below)
- Maps CEF severity to OTLP severity: 0–3 / Low → INFO, 4–6 / Medium → WARN, 7–8 / High → ERROR, 9–10 / Very-High → FATAL
- If a log has no timestamp, uses the observed time
- Leaves non-CEF lines untouched (no CEF attributes, original body kept)
- Exports via OTLP/gRPC to `127.0.0.1:4317` (insecure TLS, suitable for a local backend such as Netdata)

### How to use it

1. Open the add-on **Configuration** tab.
2. Copy the contents of [`examples/unifi-cef-syslog.yaml`](examples/unifi-cef-syslog.yaml) into the `config` option (replace the default YAML).
3. Set `location` under the syslog receiver to your timezone (the example uses `UTC` as a placeholder).
4. Point the UniFi Dream Machine **SIEM / syslog** destination at `<HA host>:5140` (UDP).
5. Point the exporter `endpoint` at your OTLP backend if it is not `127.0.0.1:4317`.
6. Save and restart the add-on.

### Attribute naming

CEF header fields become:

- `cef.version`
- `cef.device_vendor`
- `cef.device_product`
- `cef.device_version`
- `cef.signature_id`
- `cef.name`
- `cef.severity`

Each CEF extension key becomes `cef.ext.<key>`.

### Known quirks

- Syslog `appname` for these messages is typically `CEF`.
- Quotes around extension values are stripped.
- If a value contains the pattern ` word=` (space, token, equals), the parser splits there. That is an inherent CEF key-value ambiguity, not specific to this example.
- `location: UTC` is a placeholder; set it to the timezone that matches timestamps on the UniFi device.
