<div align="center">

<br/>

```
    __  _________________  ____  __________  ________ 
   /  |/  /  _/ ____/ __ \/ __ \/ ____/ __ \/  _/ __ \
  / /|_/ // // /   / /_/ / / / / / __/ /_/ // // / / /
 / /  / // // /___/ _, _/ /_/ / /_/ / _, _// // /_/ / 
/_/  /_/___/\____/_/ |_|\____/\____/_/ |_/___/_____/
```

<h3>Renewable Microgrid Arbitrage & Curtailment Dispatch</h3>

<br/>

[![n8n](https://img.shields.io/badge/Built%20on-n8n-FF6584?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io/)
[![Status](https://img.shields.io/badge/Status-Inactive%20Blueprint-8E75B2?style=flat-square)](https://n8n.io/)
[![Nodes](https://img.shields.io/badge/Nodes-25%20Nodes-1C3A5F?style=flat-square)](https://n8n.io/)
[![License](https://img.shields.io/badge/License-MIT-F7DF1E?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-00C48C?style=flat-square)](http://makeapullrequest.com)

<br/>

[**Quick Start**](#-installation) | [**Architecture**](#-architecture) | [**Node Inventory**](#-node-inventory) | [**Usage**](#-usage-examples)

<br/>

</div>

---

## What is this?

An automated n8n workflow for **Renewable Microgrid Arbitrage & Curtailment Dispatch**. It processes incoming events, transforms data payloads, and handles conditional dispatch to downstream services.

| | Component | Purpose |
|---|---|---|
| **📥** | **StickyNote_Overview** | Ingests incoming webhooks or scheduled telemetry payloads |
| **🧠** | **StickyNote_Ingestion** | Evaluates logic conditions and enriches message data |
| **🚨** | **ScheduleTrigger** | Dispatches notifications and updates database records |

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
Supports incoming webhooks and scheduled cron jobs for automatic background processing.

---

**⚡ &nbsp;Data Normalization**  
Standardizes raw input fields before forwarding payloads to analytics databases.

---

**🔒 &nbsp;Error Handling**  
Catches execution exceptions to prevent failed runs from stopping pipeline flow.

</td>
<td width="50%" valign="top">

**🧠 &nbsp;Conditional Logic**  
Filters high-priority alerts so team members only receive urgent notifications.

---

**📊 &nbsp;System Synchronization**  
Keeps external databases, logs, and notification channels in sync.

---

**🔌 &nbsp;Easy Import**  
Import the blueprint JSON directly into your n8n workspace to get started.

</td>
</tr>
</table>

---

## 🔧 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Orchestration | [n8n](https://n8n.io/) | Workflow engine, cron scheduling, webhook routing |
| Execution Engine | Node.js / JavaScript | Payload parsing and custom data mapping |
| Transport Protocol | Webhook / REST APIs | API requests and notification delivery |
| Blueprint Format | JSON (n8n v1+) | Portable workflow definition file |

---

## ✅ Prerequisites

- **n8n instance** (self-hosted or [n8n Cloud](https://app.n8n.cloud))
- Relevant API credentials configured inside your n8n workspace

---

## ⚙️ Installation

### 1. Import the Workflow

```
Workflows -> Import from File -> workflow.json
```

### 2. Configure Credentials

```
+---------------------+-------------------+--------------------------------------+
| Credential          | Type              | Attach To                            |
+---------------------+-------------------+--------------------------------------+
| API / Webhook Keys  | HTTP / OAuth2     | Integration Nodes                    |
+---------------------+-------------------+--------------------------------------+
```

### 3. Activate Workflow

```
Workflows -> [Renewable Microgrid Arbitrage & Curtailment Dispatch] -> Toggle Active
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

### cURL: Trigger Webhook

```bash
curl -X POST https://your-n8n-instance.com/webhook/microgrid-arbitrage-curtailment-dispatch \
  -H "Content-Type: application/json" \
  -d '{"timestamp": "2026-09-15T12:00:00Z", "event": "HEALTH_CHECK"}'
```

### Python: Send Event

```python
import requests

url = "https://your-n8n-instance.com/webhook/microgrid-arbitrage-curtailment-dispatch"
payload = {"event": "HEALTH_CHECK", "source": "python_script"}

res = requests.post(url, json=payload)
print("Response code:", res.status_code)
print("Data:", res.json())
```

---

## 📂 Project Structure

```
microgrid-arbitrage-curtailment-dispatch/
├── workflow.json      # Complete importable workflow definition
├── LICENSE            # MIT License file
└── README.md          # Project documentation
```

---

## 🤝 Contributing

Pull requests and issues are welcome.

```bash
# 1. Clone the repository
git clone https://github.com/abderrahman-ai/microgrid-arbitrage-curtailment-dispatch.git

# 2. Create your branch
git checkout -b patch/improvements

# 3. Commit your changes
git commit -m "docs: refine workflow description and node names"

# 4. Push to origin
git push origin patch/improvements
```

---

## 📄 License

Released under the **MIT License**. Check [`LICENSE`](LICENSE) for full details.

---

<div align="center">

<br/>

Built with [n8n](https://n8n.io/)

<br/>

**[Back to top](#)**

</div>
