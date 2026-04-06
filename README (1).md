# SDN Port Status Monitoring Tool
**Project 13 | UE24CS252B – Computer Networks | PES University**

---

## Problem Statement

In Software Defined Networks (SDN), switches are controlled by a centralized controller. When a physical or virtual port on a switch changes state (goes UP or DOWN), the network must detect and respond to it immediately. This project implements a **Port Status Monitoring Tool** using Mininet and the POX OpenFlow controller that:

- Detects port UP/DOWN events in real time
- Logs all changes with timestamps to a file
- Generates alerts when ports fail
- Displays a live port status table per switch
- Acts as a learning switch for packet forwarding

---

## Architecture

```
    h1 (10.0.0.1) ─┐
    h2 (10.0.0.2) ─┴──── s1 ──── s2 ────┬── h3 (10.0.0.3)
                                          └── h4 (10.0.0.4)

    s1, s2 → OVS Switches (OpenFlow 1.0)
    POX Controller → Listens on port 6633
```

- **2 switches** (s1, s2) connected to each other
- **4 hosts** (h1–h4) with static IPs
- **POX controller** handles all OpenFlow events

---

## Tech Stack

| Component | Tool |
|-----------|------|
| Network Emulator | Mininet 2.3.0 |
| SDN Controller | POX 0.7.0 (gar) |
| Protocol | OpenFlow 1.0 |
| OS | Ubuntu 24.04 (ARM64) |
| Validation Tools | iperf, tshark, ovs-ofctl |

---

## Setup & Installation

### Prerequisites

```bash
sudo apt update && sudo apt upgrade -y --fix-missing
sudo apt install mininet git python3 wireshark tshark iperf net-tools -y
```

### Install POX

```bash
cd ~
git clone https://github.com/noxrepo/pox
cd pox
python3 pox.py --version   # Should print: POX 0.7.0 (gar)
```

### Clone This Repo

```bash
git clone https://github.com/YOUR_USERNAME/sdn-port-monitor.git
cd sdn-port-monitor
cp port_monitor.py ~/pox/ext/
```

---

## Running the Project

> You need **two terminals** open simultaneously.

### Terminal 1 — Start POX Controller

```bash
cd ~/pox
python3 pox.py openflow.of_01 --port=6633 port_monitor
```

Wait until you see:
```
PortMonitor module loaded. Waiting for switches...
INFO:core:POX 0.7.0 (gar) is up.
```

### Terminal 2 — Start Mininet Topology

```bash
sudo python3 topology.py
```

The topology will automatically run 4 test scenarios, then drop into the Mininet CLI.

---

## Test Scenarios

### Automated (run on startup)

| Scenario | What Happens | Expected Controller Output |
|----------|-------------|---------------------------|
| TEST 1 | `pingall` — all 4 hosts ping each other | 0% packet loss |
| TEST 2 | h2 ↔ s1 link brought DOWN | `⚠️ ALERT: Port 2 went DOWN` |
| TEST 3 | h2 ↔ s1 link brought back UP | `✅ RECOVERY: Port 2 is back UP` |
| TEST 4 | `pingall` again after recovery | 0% packet loss |

### Manual (in Mininet CLI)

```bash
# Bring a port down
mininet> link s1 h1 down

# Bring it back up
mininet> link s1 h1 up

# Test connectivity
mininet> h1 ping h3 -c 5

# Throughput test
mininet> iperf h1 h3

# View flow table
mininet> sh ovs-ofctl dump-flows s1

# View port status
mininet> sh ovs-ofctl show s1
```

---

## Expected Output

### Controller (Terminal 1)
```
[2026-04-06 08:25:01] Switch CONNECTED: DPID=00-00-00-00-00-01
[2026-04-06 08:25:01] --- Initial Port Status for Switch 00-00-00-00-00-01 ---
[2026-04-06 08:25:01]   ✅ Port 1 (s1-eth1) = UP
[2026-04-06 08:25:01]   ✅ Port 2 (s1-eth2) = UP
[2026-04-06 08:25:01]   ✅ Port 3 (s1-eth3) = UP

[2026-04-06 08:25:04] PORT STATUS CHANGE | Switch=00-00-00-00-00-01 | Port=2 (s1-eth2) | Status=DOWN
[2026-04-06 08:25:04] ⚠️  ALERT: Port 2 (s1-eth2) on Switch 00-00-00-00-00-01 went DOWN!

📊 Port Status Table — Switch 00-00-00-00-00-01:
  ✅ Port 1: UP
  ❌ Port 2: DOWN
  ✅ Port 3: UP

[2026-04-06 08:25:07] ✅  RECOVERY: Port 2 (s1-eth2) on Switch 00-00-00-00-00-01 is back UP.
```

### Mininet (Terminal 2)
```
*** Results: 0% dropped (12/12 received)   ← TEST 1 & 4
```

---

## File Structure

```
sdn-port-monitor/
├── port_monitor.py       ← POX controller (main logic)
├── topology.py           ← Mininet topology + automated tests
├── port_status_log.txt   ← Auto-generated log file
└── README.md
```

---

## SDN Concepts Demonstrated

- **Controller–Switch interaction** via OpenFlow 1.0
- **Packet-in event handling** with MAC learning
- **Match–action flow rules** installed dynamically
- **PortStatus event** monitoring (OFPPR_ADD / DELETE / MODIFY)
- **Table-miss flow entry** (priority 0, send to controller)
- **Flow timeouts** (idle: 30s, hard: 120s)

---

## Cleanup

```bash
# Stop Mininet and clean up
sudo mn -c

# Stop POX
Ctrl+C in Terminal 1
```

---

## References

1. Mininet Documentation — https://mininet.org/overview/
2. POX Wiki — https://noxrepo.github.io/pox-doc/html/
3. OpenFlow 1.0 Specification — https://opennetworking.org/
4. Mininet Walkthrough — https://mininet.org/walkthrough/
5. OVS Documentation — https://docs.openvswitch.org/
