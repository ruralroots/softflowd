# Softflowd/Softflowctl – ***NetFlow™*** Flow Processor & Exporter

> A software implementation of Cisco's NetFlow™ traffic accounting protocol for OpenWrt routers

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration](#configuration)
- [Runtime Commands](#runtime-commands)
- [Softflowctl Usage](#softflowctl-usage)
- [Credits](#credits)

---

## Overview

Softflowd is a software implementation of Cisco's *NetFlow™* traffic accounting protocol designed for minimal CPU overhead. It collects and tracks traffic flows by listening on promiscuous interfaces, making it ideal for network monitoring and analysis.

### Key Capabilities

- **Flow Tracking**: Semi-statefully tracks traffic flows on network interfaces
- **NetFlow Export**: Exports flow data via NetFlow to a collecting host for analysis
- **PCAP Support**: Reads, analyzes, and exports pcap packet capture files
- **Job Control**: Softflowctl companion program for runtime management via control sockets
- **OpenWrt Integration**: Full integration with OpenWrt router operating system
- **Flexible Sampling**: Configurable sampling rates and filtering options

### Supported NetFlow Versions

Softflowd supports NetFlow versions **1, 5, and 9**, with IPv6 support in NetFlow V9.

### Collector Compatibility

Any NetFlow-compatible collector works with Softflowd. Popular options include commercial solutions and open-source tools. Search for *NetFlow Analysis software* to find collectors for your platform.

---

## Quick Start

For experienced users, here's the minimal setup:

```bash
# 1. Update repository and install
opkg update
opkg install softflowd

# 2. Download and deploy configuration files
# (Use SCP or web interface to copy files)

# 3. Start monitoring
/etc/init.d/softflowd start

# 4. Check active interfaces
/etc/init.d/softflowd active
```

For detailed setup, see [Installation](#installation).

---

## Installation

### Prerequisites

- OpenWrt-based router
- Network interface(s) available for monitoring
- NetFlow collector (optional, for flow export)

### Step-by-Step Installation

1. **Update your local opkg repository:**
   ```bash
   opkg update
   ```

2. **Install Softflowd binaries:**
   ```bash
   opkg install softflowd
   ```
   This installs both Softflowd and Softflowctl binaries.

3. **Deploy configuration files:**
   - Download this repository (Code → Download ZIP)
   - Extract the files
   - Copy to your router using SCP:

   ```bash
   scp softflowd.init root@router:/etc/init.d/softflowd
   scp softflowd.hotplug root@router:/etc/hotplug.d/iface/40-softflowd
   scp softflowd.config root@router:/etc/config/softflowd
   ```

4. **Start Softflowd:**
   ```bash
   /etc/init.d/softflowd start
   ```

### Create Command Alias (Optional)

Add this to `/etc/profile` or `.bashrc` for convenient shortcuts:

```bash
alias sf='/etc/init.d/softflowd'
```

Then use commands like: `sf statistics wan`

---

## Configuration

### Configuration File Location

**`/etc/config/softflowd`**

Each monitored interface is defined in a `config ctlsock` section, which declares:
- The interface to monitor
- The control socket file name
- Export destinations and parameters

### Performance Considerations

⚠️ **Warning**: Softflowd can place significant load on your router and collector host, especially when:
- Running multiple instances
- Operating in high-traffic environments
- Tracking many flows with detailed sampling

Use caution when configuring sampling rates and the maximum number of concurrent flows.

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| **enabled** | `1` | Enable (1) or disable (0) interface monitoring |
| **interface** | — | Interface name to monitor (e.g., `wan`, `lan`, `eth0`) |
| **pcap_file** | — | Full path to pcap packet capture file to analyze. Softflowd processes the entire file and exits after displaying summary statistics. **Note**: Either `host_port` OR `pcap_file` must be specified, not both. |
| **timeout1–timeout8** | — | Custom timeout values (see `softflowd(8)` man page). Overrides default timeouts for specific flow types. |
| **max_flows** | `8192` | Maximum concurrent flows. When exceeded, least recently seen flows are forcibly expired. Increase for high-traffic environments. |
| **host_port** | — | Collector address in format `IP:Port`. Specifies the NetFlow collector host. **Note**: Either `host_port` OR `pcap_file` must be specified, not both. |
| **export_version** | `9` | NetFlow version: `1`, `5`, or `9`. Use `9` for IPv6 support. |
| **hoplimit** | — | IPv4 TTL or IPv6 hop limit for exported flows. Defaults to system TTL for unicast export. |
| **tracking_level** | `full` | Flow tracking detail: `full` (all fields), `proto` (source, destination, protocol), or `ip` (source and destination only). |
| **track_ipv6** | `0` | Set to `1` to enable IPv6 tracking regardless of interface support. Mainly for debugging. |
| **sampling_rate** | `100` | Sampling denominator. For example, `100` means sample 1 of every 100 packets. Higher values reduce data volume. |
| **bpf_filter** | — | Berkeley Packet Filter expression to limit monitored traffic (see [BPF documentation](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter)). Example: `tcp port 80` |

### Configuration Example

```
config ctlsock 'wan_monitor'
	option enabled '1'
	option interface 'wan'
	option host_port '192.168.1.100:2055'
	option export_version '9'
	option max_flows '16384'
	option sampling_rate '100'
	option tracking_level 'full'

config ctlsock 'lan_monitor'
	option enabled '1'
	option interface 'lan'
	option pcap_file '/tmp/capture.pcap'
	option export_version '9'
```

---

## Runtime Commands

### Command Syntax

```
/etc/init.d/softflowd [command] [ctlsock] [bool]
```

Or using the alias: `sf [command] [ctlsock] [bool]`

Omitting the command will display available commands.

### Available Commands

| Command | Arguments | Description |
|---------|-----------|-------------|
| **start** | — | Start Softflowd service if not configured for auto-start. |
| **statistics** | `[ctlsock]` | Show interface statistics and flow counts. |
| **dump** | `[ctlsock]` | Dump all active flows for an interface and resume monitoring. Reported only—no processing. |
| **pause** | `[ctlsock]` | Temporarily pause flow collection on an interface. |
| **resume** | `[ctlsock]` | Resume flow collection on a paused interface. |
| **expire** | `[ctlsock]` | Expire all flows for an interface and process/export the data. |
| **delete** | `[ctlsock]` | Delete all flows for an interface without processing or exporting. |
| **timeouts** | `[ctlsock]` | Show current timeout settings for an interface. |
| **active** | — | List all active interfaces with their process IDs. |
| **shutdown** | — | Gracefully exit all instances, close Softflowd, and clean up the environment. All active flows are expired, processed, and exported. |
| **update** | `[ctlsock] [0\|1]` | Enable (1) or disable (0) interface monitoring and restart. Interfaces must be pre-configured. |
| **debugUp** | `[ctlsock]` | Increase debug verbosity for an instance. Persists only while the instance is active. |
| **debugDown** | `[ctlsock]` | Decrease debug verbosity for an instance. Persists only while the instance is active. |

---

## Softflowctl Usage

This section provides practical examples using the alias notation. See [Quick Start](#quick-start) if you haven't created the alias yet.

### View Statistics

Show real-time statistics for a monitored interface:

```bash
sf statistics wan
# Output: Interface statistics for the 'wan' interface
```

### Dump Active Flows

Export all currently active flows without disrupting monitoring:

```bash
sf dump wan
# Output: All active flows for 'wan' interface (data reported, not processed)
```

### Pause and Resume Monitoring

Temporarily stop monitoring on an interface:

```bash
sf pause wan
# Flow monitoring pauses

sf resume wan
# Flow monitoring resumes
```

### Expire Flows

Force expiration of active flows and process/export the data:

```bash
sf expire wan
# All active 'wan' flows are processed and exported
```

### Delete Flows

Discard active flows without processing:

```bash
sf delete wan
# All flows discarded (no export)
```

### View Timeout Configuration

Display timeout settings:

```bash
sf timeouts wan
# Output: Current timeout values for 'wan' interface
```

### List Active Interfaces

Show all monitored interfaces and their process IDs:

```bash
sf active
# Output: wan (PID: 1234), lan (PID: 1235), ...
```

### Graceful Shutdown

Exit all instances and clean up:

```bash
sf shutdown
# All instances exit gracefully, flows are expired/exported, environment cleaned
```

To **immediately terminate** without exporting flows:

```bash
sf stop && sf shutdown
# Equivalent to `exit` command in softflowctl
```

### Adjust Debug Verbosity

Increase debug output:

```bash
sf debugUp wan
# Debug verbosity for 'wan' instance increased
```

Decrease debug output:

```bash
sf debugDown vpn0
# Debug verbosity for 'vpn0' instance decreased
```

**Note**: Debug settings persist only while the instance is active.

### Enable/Disable Interfaces at Runtime

Enable a disabled interface:

```bash
sf update wan 1
# 'wan' interface enabled and monitoring starts
```

Disable an active interface:

```bash
sf update vpn0 0
# 'vpn0' interface disabled and monitoring stops gracefully
```

**Important**: Interfaces must be pre-configured in `/etc/config/softflowd`. The `update` command only enables/disables existing configurations. To make changes permanent, edit the config file directly. Hotplug events trigger Softflowd restart with your config.

---

## Documentation

For detailed information, see the Linux man pages:

- **`man softflowd(8)`** - Main Softflowd daemon documentation
- **`man softflowctl(8)`** - Softflowctl control program documentation
- **`man bpf(4)`** - Berkeley Packet Filter documentation

---

## Credits

**Original Creator:**
- Damien Miller <djm@mindrot.org>
- Developer website: http://mindrot.org

**Current Maintainer:**
- Hitoshi Irino <irino@sfc.wide.ad.jp>
- Repository: https://github.com/irino/softflowd/
- Issues: https://github.com/irino/softflowd/issues

**OpenWrt Integration & This Repository:**
- ruralroots

---

## License

See LICENSE file in repository for details.

---

**Last Updated**: 2026-05-10
