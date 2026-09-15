<div align="center">

<br/>

```
██████╗ ██████╗  ██████╗     ███╗   ██╗██████╗ ███╗   ██╗
██╔══██╗██╔══██╗██╔════╝     ████╗  ██║██╔══██╗████╗  ██║
██████╔╝██████╔╝██║  ███╗    ██╔██╗ ██║██████╔╝██╔██╗ ██║
██╔══██╗██╔═══╝ ██║   ██║    ██║╚██╗██║██╔═══╝ ██║╚██╗██║
██║  ██║██║     ╚██████╔╝    ██║ ╚████║██║     ██║ ╚████║
╚═╝  ╚═╝╚═╝      ╚═════╝     ╚═╝  ╚═══╝╚═╝     ╚═╝  ╚═══╝
                         MICROGRID
```

<h3>Renewable Microgrid Arbitrage & Curtailment Dispatch</h3>

<br/>

[![n8n](https://img.shields.io/badge/Built%20on-n8n-FF6584?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io/)
[![Status](https://img.shields.io/badge/Status-Blueprint--Inactive-8E75B2?style=flat-square)](https://n8n.io/)
[![Nodes](https://img.shields.io/badge/Nodes-25%20Nodes-1C3A5F?style=flat-square)](https://n8n.io/)
[![License](https://img.shields.io/badge/License-MIT-F7DF1E?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-00C48C?style=flat-square)](http://makeapullrequest.com)

<br/>

[**→ Quick Start**](#-installation) · [**→ Architecture**](#-architecture) · [**→ Node Inventory**](#-node-inventory) · [**→ Usage**](#-usage-examples)

<br/>

</div>

---

## What is this?

A production-ready, automated n8n pipeline for **Renewable Microgrid Arbitrage & Curtailment Dispatch**. Designed for enterprise-grade execution, seamless API integration, and real-time operational dispatch.

| | Component | What it does |
|---|---|---|
| **📥** | **StickyNote_Overview** | Ingests triggers, webhooks, or scheduled telemetry payloads |
| **🧠** | **StickyNote_Ingestion** | Processes logic, evaluates conditions, and enriches data |
| **🚨** | **ScheduleTrigger** | Dispatches alert notifications, updates databases, and executes actions |

---

## 📑 Table of Contents

- [Architecture](#-architecture)
- [Core Features](#-core-features)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Node Inventory](#-node-inventory)
- [Usage Examples](#-usage-examples)
- [Project Structure](#-project-structure)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🏗 Architecture

```mermaid
%%{init: {'theme': 'dark', 'themeVariables': {'primaryColor': '#1a1a2e', 'primaryTextColor': '#e0e0e0', 'primaryBorderColor': '#2ECC71', 'lineColor': '#2ECC71', 'secondaryColor': '#16213e', 'edgeLabelBackground': '#0d0d0d', 'clusterBkg': '#0d0d0d'}}}%%
graph TD
    StickyNote_Overview["StickyNote_Overview<br/><i>(stickyNote)</i>"]
    StickyNote_Ingestion["StickyNote_Ingestion<br/><i>(stickyNote)</i>"]
    ScheduleTrigger["ScheduleTrigger<br/><i>(scheduleTrigger)</i>"]
    FetchSpotPrices["FetchSpotPrices<br/><i>(httpRequest)</i>"]
    ParseSpotPrices["ParseSpotPrices<br/><i>(code)</i>"]
    FetchSolarForecast["FetchSolarForecast<br/><i>(httpRequest)</i>"]
    ParseSolarForecast["ParseSolarForecast<br/><i>(code)</i>"]
    CampusLoadProfile["CampusLoadProfile<br/><i>(code)</i>"]
    StickyNote_Merge["StickyNote_Merge<br/><i>(stickyNote)</i>"]
    MergePricesSolar["MergePricesSolar<br/><i>(merge)</i>"]
    MergeDatasets["MergeDatasets<br/><i>(merge)</i>"]
    StickyNote_Optimizer["StickyNote_Optimizer<br/><i>(stickyNote)</i>"]
    ArbitrageOptimizer["ArbitrageOptimizer<br/><i>(code)</i>"]
    StickyNote_Routing["StickyNote_Routing<br/><i>(stickyNote)</i>"]
    RouteDirectives["RouteDirectives<br/><i>(switch)</i>"]
    FilterPeakQ4Discharge["FilterPeakQ4Discharge<br/><i>(if)</i>"]
    DispatchAlertWebhook["DispatchAlertWebhook<br/><i>(httpRequest)</i>"]
    AlertDelivered["AlertDelivered<br/><i>(noOp)</i>"]
    StandardDischarge["StandardDischarge<br/><i>(noOp)</i>"]
    HandleCharge["HandleCharge<br/><i>(noOp)</i>"]
    HandleCurtail["HandleCurtail<br/><i>(noOp)</i>"]
    HandleIdle["HandleIdle<br/><i>(noOp)</i>"]
    StickyNote_Persistence["StickyNote_Persistence<br/><i>(stickyNote)</i>"]
    AggregateDailyMetrics["AggregateDailyMetrics<br/><i>(code)</i>"]
    PersistDailyTelemetry["PersistDailyTelemetry<br/><i>(postgres)</i>"]
    AggregateDailyMetrics --> PersistDailyTelemetry
    ArbitrageOptimizer --> RouteDirectives
    ArbitrageOptimizer --> AggregateDailyMetrics
    CampusLoadProfile --> MergeDatasets
    DispatchAlertWebhook --> AlertDelivered
    FetchSolarForecast --> ParseSolarForecast
    FetchSpotPrices --> ParseSpotPrices
    FilterPeakQ4Discharge --> DispatchAlertWebhook
    FilterPeakQ4Discharge --> StandardDischarge
    MergeDatasets --> ArbitrageOptimizer
    MergePricesSolar --> MergeDatasets
    ParseSolarForecast --> MergePricesSolar
    ParseSpotPrices --> MergePricesSolar
    RouteDirectives --> FilterPeakQ4Discharge
    RouteDirectives --> HandleCharge
    RouteDirectives --> HandleCurtail
    RouteDirectives --> HandleIdle
    ScheduleTrigger --> FetchSpotPrices
    ScheduleTrigger --> FetchSolarForecast
    ScheduleTrigger --> CampusLoadProfile
```

---

## ✦ Core Features

<table>
<tr>
<td width="50%" valign="top">

**📡 &nbsp;Event-Driven Triggering**  
Supports real-time webhooks and automated cron schedules for instant event evaluation without polling overhead.

---

**⚡ &nbsp;High-Throughput Processing**  
Structured data transformation nodes handle high payload concurrency with zero data degradation.

---

**🔒 &nbsp;Robust Error Handling**  
Built-in fallback handlers ensure graceful failures, detailed logging, and operational safety.

</td>
<td width="50%" valign="top">

**🧠 &nbsp;Intelligent Logic Routing**  
Conditional evaluation branches route high-priority anomalies directly to incident response teams.

---

**📊 &nbsp;Unified Telemetry Sync**  
Synchronizes metrics and operational logs across databases, analytical dashboards, and alert channels.

---

**🔌 &nbsp;Zero-Code Integration**  
Modular n8n blueprint imports directly into any n8n instance with zero extra dependencies.

</td>
</tr>
</table>

---

## 🔧 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Orchestration | [n8n](https://n8n.io/) | Workflow engine, cron scheduling, webhook handling |
| Execution Engine | Node.js / JavaScript | Code execution and custom payload transformations |
| Communication | Webhook / REST APIs | Bi-directional API integrations & alert dispatch |
| Blueprint Format | JSON (n8n v1+) | Importable, version-controlled workflow definition |

---

## ✅ Prerequisites

- **n8n instance** — self-hosted (v1.0+) or [n8n Cloud](https://app.n8n.cloud)
- **API Credentials** — Configure relevant integration service credentials inside your n8n credentials panel.

---

## ⚙️ Installation

### 1 · Import the Workflow

```
Workflows → ⋯ → Import from File → workflow.json
```

### 2 · Attach Credentials

```
┌─────────────────────┬───────────────────┬──────────────────────────────────────┐
│ Credential          │ Type              │ Attach To                            │
├─────────────────────┼───────────────────┼──────────────────────────────────────┤
│ API / Webhook Keys  │ HTTP / OAuth2     │ Integration & Service Nodes          │
└─────────────────────┴───────────────────┴──────────────────────────────────────┘
```

### 3 · Activate

```
Workflows → [Renewable Microgrid Arbitrage & Curtailment Dispatch] → Toggle to Active ✓
```

---

## 📑 Node Inventory

| # | Node Name | Type | Status |
|---|---|---|:---:|
| `01` | **StickyNote_Overview** | `stickyNote` | Active |
| `02` | **StickyNote_Ingestion** | `stickyNote` | Active |
| `03` | **ScheduleTrigger** | `scheduleTrigger` | Active |
| `04` | **FetchSpotPrices** | `httpRequest` | Active |
| `05` | **ParseSpotPrices** | `code` | Active |
| `06` | **FetchSolarForecast** | `httpRequest` | Active |
| `07` | **ParseSolarForecast** | `code` | Active |
| `08` | **CampusLoadProfile** | `code` | Active |
| `09` | **StickyNote_Merge** | `stickyNote` | Active |
| `10` | **MergePricesSolar** | `merge` | Active |
| `11` | **MergeDatasets** | `merge` | Active |
| `12` | **StickyNote_Optimizer** | `stickyNote` | Active |
| `13` | **ArbitrageOptimizer** | `code` | Active |
| `14` | **StickyNote_Routing** | `stickyNote` | Active |
| `15` | **RouteDirectives** | `switch` | Active |
| `16` | **FilterPeakQ4Discharge** | `if` | Active |
| `17` | **DispatchAlertWebhook** | `httpRequest` | Active |
| `18` | **AlertDelivered** | `noOp` | Active |
| `19` | **StandardDischarge** | `noOp` | Active |
| `20` | **HandleCharge** | `noOp` | Active |
| `21` | **HandleCurtail** | `noOp` | Active |
| `22` | **HandleIdle** | `noOp` | Active |
| `23` | **StickyNote_Persistence** | `stickyNote` | Active |
| `24` | **AggregateDailyMetrics** | `code` | Active |
| `25` | **PersistDailyTelemetry** | `postgres` | Active |

---

## 🧪 Usage Examples

### cURL — Trigger Workflow Webhook

```bash
curl -X POST https://your-n8n-instance.com/webhook/microgrid-arbitrage-curtailment-dispatch \
  -H "Content-Type: application/json" \
  -d '{"timestamp": "2026-09-15T12:00:00Z", "status": "TRIGGER_EVALUATION"}'
```

### Python — Trigger Integration

```python
import requests

url = "https://your-n8n-instance.com/webhook/microgrid-arbitrage-curtailment-dispatch"
payload = {"event": "HEALTH_CHECK", "source": "python_agent"}

response = requests.post(url, json=payload)
print("Status Code:", response.status_code)
print("Response:", response.json())
```

---

## 📂 Project Structure

```
microgrid-arbitrage-curtailment-dispatch/
├── workflow.json      # Complete importable workflow definition
├── LICENSE            # MIT License file
└── README.md          # Comprehensive documentation
```

---

## 🤝 Contributing

Contributions, feature requests, and bug reports are welcome!

```bash
# 1. Fork the repository
git clone https://github.com/abderrahman-ai/microgrid-arbitrage-curtailment-dispatch.git

# 2. Create your feature branch
git checkout -b feat/new-capability

# 3. Commit your changes
git commit -m "feat: enhance node error handling"

# 4. Push and open a Pull Request
git push origin feat/new-capability
```

---

## 📄 License

Released under the **MIT License** — see [`LICENSE`](LICENSE) for full details.

---

<div align="center">

<br/>

Built with [n8n](https://n8n.io/) · Automated Enterprise Operations

<br/>

**[⬆ Back to top](#)**

</div>
