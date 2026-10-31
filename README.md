# OT Network Security Operations Dashboard

> **A standalone, browser-based OT/ICS network traffic analyser for security auditors.**  
> One HTML file. No installation. No backend. No internet connection required.

![License: Personal v1.0](https://img.shields.io/badge/License-Christophe%20Prince%20Personal%20v1.0-blue.svg)
![Version](https://img.shields.io/badge/version-0.11-brightgreen)
![Protocols](https://img.shields.io/badge/protocols-Modbus%20%7C%20S7%20%7C%20DNP3%20%7C%20EtherNet%2FIP%20%7C%20FINS%20%7C%20MELSEC-orange)
![Standard](https://img.shields.io/badge/standard-IEC%2062443-red)

---

## What is it?

A single `.html` file you open in Chrome or Edge. Drop a `.pcap` or `.pcapng` capture file and get instant, deep visibility into your industrial network — device fingerprinting, critical command detection, inter-VLAN traffic analysis, behavioral anomalies, and a professional PDF audit report aligned with IEC 62443.

Designed for use during **OT/ICS security audit missions** — no infrastructure, no cloud, no vendor dependency.

---

## Quick Start

1. Download `ot_dashboard.html`
2. Open it in **Chrome** or **Edge**
3. Click **Open file** or drag & drop a `.pcap` / `.pcapng` file
4. Click **Demo** to explore with built-in sample data (no file needed)

> **Large files**: processed in 64 MB chunks via a background Web Worker — the UI stays responsive even for captures exceeding 800 MB.

---

## Features

### 🔍 Deep Packet Inspection (DPI)

Full application-layer parsing for industrial protocols — not just port detection:

| Protocol | Port | What is extracted |
|---|---|---|
| **Modbus TCP** | 502 | Function Code, Unit ID, register address & value, write counts |
| **S7 / S7COMM** | 102 | TPKT/COTP/S7: program download (0x1A–0x1C) and upload (0x1D–0x1F), PLC Control (0x28, service `_INSE` / `_DELE` / `P_PROGRAM`), PLC Stop (0x29), Write Var (0x05), PDU size |
| **DNP3** | 20000 | Function Code, source/destination addresses, control commands |
| **EtherNet/IP** | 44818 | List Identity (command 0x0063): Vendor ID, Device Type, **Product Name** (as reported by the device), revision, serial number; CIP Write Tag / Reset / Start / Stop requests |
| **FINS (Omron)** | 9600 | Command codes, CPU Stop/Start, Memory Write |
| **MELSEC (Mitsubishi)** | 5006/5007 | Protocol detection, vendor identification |

### 🏭 Device Fingerprinting

Automatic identification of devices from **EtherNet/IP List Identity** responses (ODVA CIP Vol. 2): vendor, device type, **product name as reported by the device**, firmware revision and serial number.

| Vendor ID | Name shown |
|---|---|
| `1` (0x0001) | Allen-Bradley / Rockwell Automation |
| `47` (0x002F) | Omron |
| `243` (0x00F3) | Schneider Electric |
| any other | `Vendor ID <n> (0x…)` — the product name sent by the device still identifies the model |

Only vendor IDs verified against ODVA records are named; extend the `ENIP_VENDORS` table in the source to add more. Identification needs List Identity responses in the capture (UDP/TCP 44818).

**S7 PDU size** (Setup Communication) is recorded, but it cannot reliably tell an S7-1200 from an S7-1500, so no model family is inferred.

### 🚨 Critical Command Detection

Alerts when dangerous OT commands are observed in the capture:

**Siemens S7** (function code read from the parameter block of Job requests)
- `Request Download` / `Download Block` / `Download Ended` (0x1A–0x1C) — program written to the PLC → **CRITICAL**
- `PLC Stop` (0x29) → **CRITICAL**
- `PLC Control` (0x28): activate block `_INSE`, delete block `_DELE`, start/stop program `P_PROGRAM` → **CRITICAL**
- `Start Upload` / `Upload` / `End Upload` (0x1D–0x1F) — program read from the PLC → **WARNING**
- `Write Var` (0x05), other PLC Control services → **WARNING**
- **Program transfer sequence**: a download followed by block activation, program start or CPU stop on the same PLC → **CRITICAL**

**Modbus TCP**
- Write Coils (FC5/15), Write Registers (FC6/16) from IT/untrusted zone → **CRITICAL**
- FC8 Diagnostics/Restart → **CRITICAL**
- FC43 Device Identification (active reconnaissance) → **WARNING**
- Register flood (>50 writes to same register) → **WARNING**
- Write register 0, value 0 (device reset/stop) → **CRITICAL**

**DNP3**
- Operate / Direct Operate / Direct Operate No Ack (FC 0x04–0x06) — physical output control → **CRITICAL**
- Cold / Warm Restart (0x0D / 0x0E), Initialize Data / Application (0x0F / 0x10) → **CRITICAL**
- Start / Stop Application (0x11 / 0x12) → **CRITICAL**

**EtherNet/IP CIP**
- Reset (0x05), Stop (0x07) → **CRITICAL**
- Write Tag (0x4D / 0x53), Start (0x06), Execute PCCC (0x4B) → **WARNING**
- Direct SendRRData / SendUnitData requests only; requests wrapped in Unconnected_Send are not unwrapped

**FINS (Omron)**
- CPU Stop/Start, Memory Write → **CRITICAL**

### 📊 Network Analysis

- **Inter-VLAN heatmap matrix** — traffic volumes between all network segments, color-coded by volume and risk
- **Purdue Model topology** — automatic zone detection (Enterprise, DMZ, Control, Field, Untrusted) from VLAN names
- **Traffic timeline** — stacked bar chart by VLAN, click to filter flows by time interval
- **Complete flow list** — all flows, no truncation, critical flows always surfaced first
- **Host inventory** — every device with type, peers, ports, services, fingerprint, alerts
- **Scope view** — a second topology limited to the whitelisted assets, for a readable picture of large captures (Topology keeps the full view)
- **Scope Analysis** — host inventory and security findings restricted to the whitelisted assets, with the out-of-scope peers each asset talks to

### 🔎 Behavioral Analysis

- Port scan / host sweep detection (≥8 distinct destinations)
- Unusual protocol in OT zone
- Asymmetric traffic / possible exfiltration (ratio >10×)
- Excessive broadcast
- Cleartext protocols (Telnet, FTP, HTTP)
- Industrial cycle anomaly (polling gap >5× normal)
- Replay attack detection (identical command >10×)
- Slow reconnaissance (spread over >5 minutes)

### 🌐 Threat Intelligence

Built-in database covering Google/Cloudflare DNS, Microsoft/Azure, AWS, GitHub, Telegram, Tor exit nodes, multicast ranges, NTP servers. Add custom entries with CIDR support — persists in browser `localStorage`.

### 📄 PDF Audit Report

Professional report generated entirely in the browser (no data leaves your machine):

1. **Cover page** — mission name, date, risk badge, key metrics
2. **Executive Summary** — 6 KPIs, network segment table with Purdue zones
3. **Inter-VLAN Traffic Matrix** — color-coded + external communications with TI classification
4. **Security Findings** — all CRITICAL/WARNING/INFO with BEHAVIORAL/FINGERPRINT badges and evidence
5. **Host Inventory** — grouped by VLAN, EtherNet/IP identity (vendor, model, revision, serial), peer list with ports/services
6. **IEC 62443 Recommendations** — personalized to detected findings, with Security Level (SL-1/2/3) and priority

---

## Multi-file & Multi-VLAN Support

Load multiple captures simultaneously — all flows are merged into a unified view. Name files with VLAN hints for automatic segment detection:

```
V10_IT.pcap
vlan20_OT.pcapng
V30_field.pcap
VLAN40_SCADA.pcapng
```

---

## Browser Compatibility

| Browser | Status |
|---|---|
| Chrome / Chromium | ✅ Fully supported |
| Edge | ✅ Fully supported |
| Firefox | ✅ Supported |
| Safari | ⚠️ Web Worker may have limitations |

---

## Supported File Formats

- `.pcap` (libpcap, little-endian and big-endian, nanosecond precision)
- `.pcapng` (Enhanced Packet Block, Simple Packet Block)
- `.cap`
- `.csv` (Wireshark export format)

---

## Privacy & Security

- **No data leaves your browser** — everything is processed locally
- **No telemetry, no analytics, no external requests**
- **No installation** — nothing written to your system
- Works fully **offline**

---

## Limitations

- IPv6 traffic not parsed
- Encrypted payloads (TLS, SSH content) — metadata only
- EtherNet/IP fingerprinting requires List Identity packets in the capture
- EtherNet/IP: only vendor IDs verified against ODVA records are named (Rockwell, Omron, Schneider); CIP services are read from direct requests only
- Protocol decoders follow the published specifications and are verified on synthetic packets; please report any mismatch seen on real captures
- S7 precise model requires SZL Read responses (rare in passive captures); the PDU size does not identify the S7 family
- pcapng streaming: chunks accumulated then parsed at end

---

## Roadmap

- [ ] Asset registry — manually add PLCs not detected automatically
- [ ] Export CSV inventory
- [ ] Searchable flow list
- [ ] pcapng full streaming support
- [ ] VLAN-based timeline colors (configurable)
- [ ] Annotations on flows and hosts

---

## Contributing

Pull requests welcome. Please open an issue first to discuss significant changes.

---

## License

Christophe Prince Personal License v1.0 — see [LICENSE](LICENSE).

Free to use, including in paid audit and consulting missions. You may not sell, rebrand, host as a service or integrate it into a commercial product, and any resale or commercial use requires consulting the author and a written agreement first.

Copyright (c) 2026 Christophe Prince

---

## Disclaimer

This tool is based on passive network traffic analysis. It does not constitute a comprehensive security assessment or penetration test. Findings represent observed network behaviour during the capture period only. This tool does not replace a formal IEC 62443 Security Level assessment performed by a qualified auditor.

---

*Built for the OT security community — free, open, offline.*
