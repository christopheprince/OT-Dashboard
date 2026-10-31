# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/); versions follow `MAJOR.MINOR.PATCH` (pre-1.0: minor = features, patch = fixes).
Change types: **Added** · **Changed** · **Fixed** · **Security**.

## [0.11.0] — 2026-10-31

### Added
- **Alternative topology views** — Topology and Scope tabs keep the current view and add three more: *Sankey* (zone-to-zone ribbons), *Bundled circle* (hierarchical edge bundling) and *Circle packing* (zone > VLAN > host). Hosts/links with problems (OT critical events, external, flagged findings) are shown in red; warnings in amber.
- **Click → Flows** — selecting a host, VLAN, zone or zone→zone ribbon filters the Flows panel accordingly; in Circle packing, clicking a bubble shows its links to the other bubbles.
- Host tooltip with a "Communicates with" list.
- **Timeline bin filters the topology** — selecting a time slice dims hosts and links with no traffic in it, in Topology, Scope and the three alternative views (previously only the Flows panel reacted).

### Fixed
- Topology was empty after loading a capture or the demo while the Topology tab was already open.

## [Unreleased]

### Changed
- Splash screen: removed the name/subtitle text under the banner image (already shown in the image).

## [0.10.0] — 2026-10-29

### Added
- **Flows protocol filter** — the filter dropdown has a *Protocol* group listing every protocol found in the capture (with flow counts); it combines with the IP filter and the sort.
- **Threat Intel browser** — built-in categories are clickable (IPs/ranges, name, risk, note), each with a one-line description, plus a search box over the whole built-in database.
- **Loopback captures** — BSD/macOS loopback (link types 0 and 108, e.g. Wireshark on `lo0`) are decoded; link types 12/14 are treated as raw IP.
- **Empty-load message** — when no flow can be extracted, a visible message says why (unsupported link type, or no IPv4 traffic).
- **Modbus/TCP on any port** — a TCP payload that is a valid MBAP frame is decoded as Modbus even on a non-standard port (e.g. 5020). This is a structural heuristic.

### Changed
- **Scope behaves like Topology** — same mouse-over card (including *Communicates with*); no second detail window on click; clicking a host filters the Flows list to that host (badge + × to clear); the IP picker and flow-row click work inside Scope.
- Loopback (`127.x`) client↔server flows are kept instead of being dropped as self-traffic.

### Fixed
- Scope mouse-over card was rendered inside the hidden Topology panel and never appeared; it is now attached to the page body.
- Modbus detection in the Web Worker was missing FC5/FC15 (write coils) and FC43 (device identification), which only the fallback parser had.
- Modbus FC15/16 replies and FC1–4 replies are no longer read as requests (no false "write" events, no bogus register addresses).

## [0.9.0] — 2026-10-07

### Fixed
- Multicast (224.0.0.0/4) and limited broadcast (255.255.255.255) are no longer classified as external / "To internet" / unknown IP, so they no longer raise external-communication alerts (e.g. 224.0.0.22, 239.255.255.250). Applied in both parsers.
- Flows IP filter: a full IPv4 typed (e.g. `200.200.200.1`) now matches exactly instead of also matching `200.200.200.142`. Partial input (`200.200.200.`, `115.20`) or a trailing `*` is a prefix match.

## [0.8.0] — 2026-10-06

### Added
- **802.1Q selector** (top bar: auto / use tags / ignore tags) and a bottom-bar notice listing ignored tags.
- **Flows IP picker** — searchable, numerically sorted list of IPs, and a *Sort: IP* option.
- **Whitelist peer reveal in Topology** — whitelisted IPs are ringed and the nodes they talk to are highlighted without hovering (including same-VLAN peers); toggle chip *Whitelist peers*. Hovered/selected nodes also show their same-VLAN links.
- In-app help entry for VLANs, 802.1Q tags and the Flows picker.

### Fixed
- 802.1Q tags found inside captures named by VLAN (`V10.pcap`, `vlan20_…`) no longer create extra VLANs; the VLAN comes from the file name unless tag mode is forced.
- The `DEMO` label, skipped-packet counter and demo whitelist are reset when a real capture is loaded; the format label is now set by the Web Worker.
- Empty placeholder VLANs are no longer created when every frame of a file is tagged.

## [0.7] — 2026

### Added
- **Deep Packet Inspection (DPI)** — full application-layer parsing for Modbus, S7, DNP3, EtherNet/IP, FINS, MELSEC
- **EtherNet/IP List Identity parsing** — vendor, device type, product name, revision and serial read from the device (ODVA CIP Vol. 2, little-endian); vendor IDs named only when verified (Rockwell 1, Omron 47, Schneider 243)
- **Critical command detection** — S7 program download (0x1A–0x1C), PLC Stop (0x29), PLC Control (0x28: `_INSE`, `_DELE`, `P_PROGRAM`), Write Var; DNP3 Operate / Direct Operate (0x04–0x06), Cold/Warm Restart, Initialize, Start/Stop Application; CIP Reset / Stop / Start / Write Tag / Execute PCCC; FINS CPU Stop/Start/Memory Write; Modbus FC8/FC43
- **S7 program transfer sequence** — a program download followed by block activation, program start or CPU stop on the same PLC
- **Scope topology and Scope Analysis** — a topology and an analysis restricted to the whitelisted assets, with the out-of-scope peers each asset talks to
- **Critical events kept outside the per-flow cap** — a critical command is no longer lost when it follows a long run of cyclic traffic
- **Hardening** — text read from the wire (EtherNet/IP product name, S7 service name) is stripped of HTML metacharacters before display
- **Web Worker streaming parser** — pcap files processed in 64 MB chunks, UI stays responsive
- **Analysis tab** — two-column deep audit view: host inventory + security findings
- **PDF audit report** — 5-section professional report with IEC 62443 recommendations
- **Threat Intelligence** — built-in database + custom entries with CIDR support, localStorage persistence
- **Behavioral detection** — port scan, host sweep, asymmetric traffic, replay attack, slow scan, industrial cycle anomaly, cleartext protocols
- **Purdue Model topology** — automatic zone detection from VLAN names
- **Traffic timeline** — stacked bar chart with bin filtering
- **Dark/light theme** — full CSS variable theming including topology canvas
- **Multi-file support** — load multiple pcap/pcapng files, all merged into unified view
- **Subnet discovery** — auto-detect /24 subnets as virtual VLANs
- **User guide** — complete in-app help accessible from splash screen and topbar

### Protocols supported
- Modbus TCP (port 502)
- S7 / S7COMM (port 102, TPKT/COTP stack)
- DNP3 (port 20000)
- EtherNet/IP (port 44818, TCP + UDP List Identity)
- FINS / Omron (port 9600)
- MELSEC / Mitsubishi (ports 5006/5007)
- OPC-UA (port 4840)
- BACnet (port 47808)

### Vendor IDs named via EtherNet/IP (verified against ODVA records)
Allen-Bradley / Rockwell Automation (1) · Omron (47) · Schneider Electric (243). Any other vendor is shown as its raw Vendor ID together with the product name reported by the device.
