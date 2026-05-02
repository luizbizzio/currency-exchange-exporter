<h1 align="center">Currency Exchange Exporter 💱</h1>

<p align="center">Prometheus exporter for currency exchange rates.</p>

It fetches exchange rates from a public dataset on an interval and exposes metrics on `/metrics`.
Prometheus scraping does not trigger extra external API requests. The exporter serves cached values until the next update cycle.

This exporter is for monitoring and dashboards. It is not for trading or real-time pricing.

## Examples (Grafana)

### Time series panel

<p align="center">
  <img src="images/metrics.png" alt="Grafana time series example" width="700"/>
</p>

### Stat panel

<p align="center">
  <img src="images/metrics2.png" alt="Grafana stat example" width="700"/>
</p>

## Contents

### 📦 Binary downloads

- 🐧 [Linux arm64 binary](#linux-arm64-)
- 🐧 [Linux amd64 binary](#linux-amd64-)
- 🪟 [Windows amd64 binary](#windows-amd64-)

### 🐳 Docker

- 🐧 [Linux/macOS Docker](#linuxmacos-)
- 🪟 [Windows PowerShell Docker](#windows-powershell-)

### 🛠️ Setup

- 🔧 [Configure `config.yaml`](#configuration-)
- ✅ [Verify exporter](#verify-)

## Features 📊

- Metrics on `/metrics`
- Pair-based config in `config.yaml`
- One snapshot request per update cycle
- Cached values between update cycles
- Cross-rate calculation from a pivot currency
- Retries with exponential backoff for temporary network issues
- Unsupported pair detection
- Health and readiness endpoints
  - `/-/healthy` or `/healthz`
  - `/-/ready` or `/readyz`
- Optional default Python and process metrics
- Docker support through GitHub Container Registry
- Prebuilt binaries for Linux and Windows

## Requirements

- Network access to the public exchange-rate dataset
- A `config.yaml` file
- One of:
  - Docker
  - A prebuilt release binary
  - Python 3.10+ for local development

## How it works

- You define currency pairs in `config.yaml`, for example `EUR-USD` or `USD-BRL`.
- The exporter chooses a pivot currency. It prefers `USD` if it appears in your pairs.
- It downloads one JSON snapshot for the pivot currency.
- It builds a lookup table from that snapshot.
- Each configured pair is calculated using cross-rate math.
- Prometheus scrapes only read cached metrics.

If a currency code does not exist in the snapshot, the exporter stays up and marks the pair as unsupported.

## Data source

- Project: https://github.com/fawazahmed0/exchange-api
- Symbols list: https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies.json

The upstream dataset updates daily. Updating more often than once per day will usually return the same values.

Your exporter refresh interval is controlled by `scrape.update_interval_seconds`.

## Install 📥

You can run this exporter with a prebuilt binary, with Docker, or from source.

Recommended options:

| Method | Best for |
|---|---|
| 📦 Binary | Raspberry Pi, Linux servers, Windows testing |
| 🐳 Docker | Servers, homelabs, containerized monitoring stacks |
| 🐍 Python source | Development and debugging |

## Option 1: Run from binary 📦

Download the binary for your system from the latest GitHub Release.

| System | Asset |
|---|---|
| 🐧 Linux amd64 | `currency-exchange-exporter-linux-amd64` |
| 🐧 Linux arm64 | `currency-exchange-exporter-linux-arm64` |
| 🪟 Windows amd64 | `currency-exchange-exporter-windows-amd64.exe` |

### Linux arm64 🐧

Use this for Raspberry Pi OS 64-bit and other Linux ARM64 systems.

```bash
mkdir -p currency-exchange-exporter
cd currency-exchange-exporter

curl -fL -o currency-exchange-exporter-linux-arm64 https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/currency-exchange-exporter-linux-arm64
curl -fL -o config.example.yaml https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/config.example.yaml

chmod +x currency-exchange-exporter-linux-arm64
cp config.example.yaml config.yaml
nano config.yaml

./currency-exchange-exporter-linux-arm64 --config-file config.yaml
```

### Linux amd64 🐧

Use this for most Linux PCs, servers, and VMs.

```bash
mkdir -p currency-exchange-exporter
cd currency-exchange-exporter

curl -fL -o currency-exchange-exporter-linux-amd64 https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/currency-exchange-exporter-linux-amd64
curl -fL -o config.example.yaml https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/config.example.yaml

chmod +x currency-exchange-exporter-linux-amd64
cp config.example.yaml config.yaml
nano config.yaml

./currency-exchange-exporter-linux-amd64 --config-file config.yaml
```

### Windows amd64 🪟

Use Windows amd64 for most Windows PCs.

Run these commands in PowerShell.

```powershell
New-Item -ItemType Directory -Force -Path currency-exchange-exporter
Set-Location currency-exchange-exporter

Invoke-WebRequest -Uri "https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/currency-exchange-exporter-windows-amd64.exe" -OutFile "currency-exchange-exporter-windows-amd64.exe"
Invoke-WebRequest -Uri "https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/config.example.yaml" -OutFile "config.example.yaml"

Copy-Item config.example.yaml config.yaml
notepad config.yaml

.\currency-exchange-exporter-windows-amd64.exe --config-file config.yaml
```
## Option 2: Run with Docker 🐳

The exporter is available on GitHub Container Registry.

The container is stateless and does not include configuration.
Create a local `config.yaml` file first, then mount it to `/config/config.yaml`.

### Linux/macOS 🐧

```bash
mkdir -p currency-exchange-exporter
cd currency-exchange-exporter

curl -fL -o config.example.yaml https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/config.example.yaml

cp config.example.yaml config.yaml
nano config.yaml

docker run -d \
  --name currency-exchange-exporter \
  -p 9131:9131 \
  -v "$(pwd)/config.yaml:/config/config.yaml:ro" \
  --restart unless-stopped \
  ghcr.io/luizbizzio/currency-exchange-exporter:latest
```

### Windows PowerShell 🪟

Run these commands in PowerShell or in Windows Terminal with a PowerShell tab.

```powershell
New-Item -ItemType Directory -Force -Path currency-exchange-exporter
Set-Location currency-exchange-exporter

Invoke-WebRequest -Uri "https://github.com/luizbizzio/currency-exchange-exporter/releases/latest/download/config.example.yaml" -OutFile "config.example.yaml"

Copy-Item config.example.yaml config.yaml
notepad config.yaml

docker run -d `
  --name currency-exchange-exporter `
  -p 9131:9131 `
  -v "${PWD}\config.yaml:/config/config.yaml:ro" `
  --restart unless-stopped `
  ghcr.io/luizbizzio/currency-exchange-exporter:latest
```

### Container health

This container includes a built-in Docker healthcheck.

- Liveness: `/-/healthy`
- Readiness: `/-/ready`

A healthy container means the exporter HTTP server is running.
It does not guarantee that the latest external rate update succeeded.

Use `currency_exchange_exporter_up` and `currency_exchange_exporter_last_success_timestamp` to validate update state.

## Option 3: Run from source 🐍

Use this for development or debugging.

```bash
git clone --depth 1 https://github.com/luizbizzio/currency-exchange-exporter.git
cd currency-exchange-exporter

python3 -m venv .venv
. .venv/bin/activate

python -m pip install -U pip
python -m pip install -r requirements.txt

cp config.example.yaml config.yaml
nano config.yaml

python currency_exchange_exporter.py --config-file config.yaml
```

## Verify ✅

The exporter listens on port `9131` by default.

- Metrics: `http://localhost:9131/metrics`
- Health: `http://localhost:9131/-/healthy`
- Ready: `http://localhost:9131/-/ready`

🐧 Linux/macOS:

```bash
curl http://localhost:9131/metrics
curl http://localhost:9131/-/healthy
curl http://localhost:9131/-/ready
```

🪟 Windows PowerShell:

```powershell
Invoke-WebRequest http://localhost:9131/metrics
Invoke-WebRequest http://localhost:9131/-/healthy
Invoke-WebRequest http://localhost:9131/-/ready
```

## Configuration 🔧

### Config file

Create a `config.yaml` file before running the exporter.

The exporter looks for `config.yaml` in the current directory by default.

You can also pass it explicitly:

```bash
./currency-exchange-exporter-linux-arm64 --config-file config.yaml
```

### Example config

Copy `config.example.yaml` to `config.yaml` and edit the currency pairs if needed.

```yaml
web:
  listen_address: "0.0.0.0:9131"
  telemetry_path: "/metrics"

scrape:
  timeout_seconds: 10
  update_interval_seconds: 28800
  retries: 3
  retry_backoff_seconds: 1
  expose_default_metrics: false
  log_level: INFO

pairs:
  - BTC-USD
  - EUR-BRL
  - USD-BRL
  - GBP-EUR
  - EUR-USD
  - CNY-USD
```

### Config notes

- Pair format is `BASE-QUOTE`, for example `USD-BRL`.
- Pair names are normalized to uppercase.
- Underscore also works, for example `USD_BRL`.
- `update_interval_seconds: 28800` means 8 hours.
- `expose_default_metrics: true` also exposes default `python_*` and `process_*` metrics.
- If a currency code does not exist in the snapshot, the exporter sets `currency_pair_supported` to `0`.
## Prometheus scrape config

Add this to your Prometheus config:

```yaml
scrape_configs:
  - job_name: "currency-exchange-exporter"
    static_configs:
      - targets: ["YOUR_EXPORTER_IP:9131"]
```

If you update only once or a few times per day, you can use a longer `scrape_interval` for this job.

## Grafana queries

Single pair:

```promql
currency_exchange_rate{base="EUR",quote="USD"}
```

Many pairs in one panel:

```promql
currency_exchange_rate{base=~"EUR|USD|GBP|CNY|BTC",quote=~"USD|EUR|BRL"}
```

Legend format:

```text
{{base}}-{{quote}}
```

## Metrics 📈

| Name | Type | Description | Scope |
|---|---|---|---|
| `currency_exchange_exporter_up` | Gauge | 1 if last update succeeded, else 0 | Global |
| `currency_exchange_exporter_last_update_timestamp` | Gauge | Unix timestamp of last update attempt | Global |
| `currency_exchange_exporter_last_success_timestamp` | Gauge | Unix timestamp of last successful update | Global |
| `currency_exchange_exporter_update_duration_seconds` | Gauge | Duration of the last update in seconds | Global |
| `currency_exchange_exporter_errors_total` | Counter | Total number of update errors | Global |
| `currency_exchange_exporter_config_invalid_pairs` | Gauge | Number of configured pairs missing in current snapshot | Global |
| `currency_exchange_exporter_invalid_pairs_total` | Counter | Number of update cycles with missing pairs | Global |
| `currency_exchange_exporter_snapshot_info` | Info | Snapshot date and pivot currency | Global |
| `currency_exchange_rate` | Gauge | Exchange rate from base to quote | Pair |
| `currency_pair_supported` | Gauge | 1 if pair can be calculated, else 0 | Pair |

## Troubleshooting 🔍

If you do not see `currency_exchange_rate`:

- Open `/metrics` and search for `currency_`.
- Confirm the exporter is running on the expected port.
- Check logs. A bad URL or timeout will set `currency_exchange_exporter_up = 0`.
- Check that `pairs` contains valid `BASE-QUOTE` entries.

If `currency_exchange_exporter_config_invalid_pairs > 0`:

- One or more codes in `pairs` do not exist in the current snapshot.
- Check the symbols list:
  https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies.json

If `/-/ready` returns `503`:

- The exporter has not completed a successful update yet.
- Check internet access from the host or container.
- Check `currency_exchange_exporter_up`.

## Security notice ⚠️

This exporter does not require API keys or credentials.

It fetches public exchange-rate data from an external public dataset.
Do not use these values for trading, accounting, tax reporting, or real-time pricing decisions.

Keep your local `config.yaml` out of Git if you customize it.

Use `config.example.yaml` in the repository and keep your real `config.yaml` local.

## License 📄

This project is licensed under the [Apache License 2.0](./LICENSE).
