---
name: ps2-elf
description: PS2 ELF reverse engineering toolkit for analyzing PlayStation 2 game executables, extracting network protocols, and building server emulators. Use for PS2 game reverse engineering and server emulation development.
license: MIT
metadata:
  author: VTSTech
  version: "0.0.1"
  repository: "https://github.com/VTSTech/skills"
---

# PS2 ELF Reverse Engineering Toolkit

## Overview

This skill provides tools and workflows for analyzing PlayStation 2 ELF files, extracting network protocols, and building server emulators for PS2 network games.

> **Note:** PS2 games typically use MIPS R3000 architecture. `objdump` has limited support for MIPS, so **radare2** is the preferred tool for disassembly and analysis.

### Case Study: NASCAR 2004 EA SPORTS

**Analysis of NASCAR.ELF (3.1 MB, MIPS R3000, stripped):**

```bash
# Quick ELF analysis
readelf -h NASCAR.ELF
readelf -S NASCAR.ELF | grep -E 'text|data|rodata|bss'

# Network protocol extraction
strings NASCAR.ELF | grep -i network
strings NASCAR.ELF | grep -E '\\d{4,5}'

# Detailed string analysis
strings NASCAR.ELF | grep -E 'recv|send|connecting'

# Radare2 analysis
r2 NASCAR.ELF -c 'iz'  # List strings
r2 NASCAR.ELF -c 'iz~network'  # Network strings
r2 NASCAR.ELF -c 'pdf @ 0x2b8b60'  # Disassemble specific function
```

**Key Findings:**
- **Entry Point:** 0x100008
- **Packet Format:** `recv: %c%c%c%c/%c%c%c%c` (8 bytes total, 4 bytes / 4 bytes)
- **Connection String:** `connecting to %08x:%d` (hex IP + decimal port)
- **Connection Types:** UDP, Reliable UDP, Escrow, FlowModule-based
- **Architecture:** FlowModule pattern (EA middleware)
- **Server Messages:** "The Server is currently down for maintenance", "server ICMP timeout"

**Note:** The protocol NASCAR uses may appear in other EA Sports titles. This analysis saved to `~/ps2-analysis/comprehensive_NASCAR.ELF_analysis.md`

## Tools Available

### 1. ELF Analysis (`/ps2-elf analyze <elf-path>`)

Analyze PS2 ELF structure, identify sections, and extract key information.

**Usage:**
```bash
/skill:ps2-elf analyze /path/to/game.elf
```

**What it does:**
- Extracts ELF header information
- Lists all sections and their properties
- Identifies entry points and code sections
- Finds string tables and symbol names
- Detects commonly used libraries (libnet, libps2ip, etc.)
- Identifies potential network-related functions
- **MIPS PS2 Specific:** Detects MIPS R3000 architecture, little-endian, System V ABI

**Example Output:**
```
=== PS2 ELF Analysis ===
File: NASCAR.ELF
Type: ELF 32-bit LSB executable, MIPS R3000, statically linked, stripped
Size: 3.1 MB
Entry Point: 0x100008
Sections: .text (2.5 MB), .data (130 KB), .rodata (290 KB), .bss (1.2 MB)
```

### 2. Network Protocol Extraction (`/ps2-elf network <elf-path>`)

Extract network protocols from the ELF file.

**Usage:**
```bash
/skill:ps2-elf network /path/to/game.elf
```

**What it does:**
- Scans for network-related functions (send, recv, connect, etc.)
- Extracts known IP addresses and port numbers
- Identifies protocol constants (port numbers, packet sizes)
- Lists potential server endpoints
- Finds string literals related to protocols
- **PS2 Specific:** Identifies FlowModule architecture, multiple connection types (UDP, Reliable UDP, Escrow)

**Example Output:**
```
=== PS2 ELF Network Protocol Extraction ===
Network-Related Strings:
- NetworkConnectionUdpC, NetworkConnectionReliableUdpC, NetworkConnectionEscrowC
- FMNetworkGameConnectionsC, FMNetOnlineC
- Network GameClient, Network GameServer
- "connecting to %08x:%d"
- "recv: %c%c%c%c/%c%c%c%c"

Connection Types:
1. NetworkConnectionC - Base network connection
2. NetworkConnectionUdpC - UDP-based
3. NetworkConnectionReliableUdpC - Reliable UDP
4. NetworkConnectionEscrowC - Transactional data
5. FMNetworkGameConnectionsC - FlowModule manager
```

### 3. String Analysis (`/ps2-elf strings <elf-path> [pattern]`)

Extract and search strings from the ELF file.

**Usage:**
```bash
/skill:ps2-elf strings /path/to/game.elf "login"
/skill:ps2-elf strings /path/to/game.elf
```

**What it does:**
- Extracts all printable strings from the ELF
- Filters by pattern if provided
- Groups strings by context (nearby strings)
- Identifies potential protocol strings, error messages, etc.
- **PS2 Specific:** Finds connection strings, packet formats, server messages, configuration tables

### 4. Disassembly Export (`/ps2-elf disasm <elf-path> [section] [address] [length]`)

Export disassembly of specific sections or functions.

**Usage:**
```bash
/skill:ps2-elf disasm /path/to/game.elf .text
/skill:ps2-elf disasm /path/to/game.elf .text 0x1000 1000
```

**What it does:**
- Uses radare2 to disassemble specified section
- Can target specific address range
- Outputs to file for further analysis
- **PS2 Specific:** MIPS R3000 disassembly with radare2 integration for advanced analysis

### 5. Build Server Template (`/ps2-elf server-template <protocol-info> [language]`)

Generate a server template based on protocol information.

**Usage:**
```bash
/skill:ps2-elf server-template "NASCAR server - 8-byte packets - UDP" python
/skill:ps2-elf server-template "PS2 game server - hex IP format" go
```

**What it does:**
- Creates a basic server skeleton
- Configures network socket
- Sets up basic protocol handling
- Includes logging and error handling
- **PS2 Specific:** Templates include hex IP parsing, 8-byte packet handling, connection timeout handling

### 6. Protocol Documentation (`/ps2-elf document-protocol <elf-path> [output-file]`)

Generate protocol documentation from ELF analysis.

**Usage:**
```bash
/skill:ps2-elf document-protocol /path/to/game.elf protocol.md
```

**What it does:**
- Creates comprehensive protocol documentation
- Lists all detected endpoints
- Documents packet structures
- Includes example packet formats
- Suggests server implementation approach
- **PS2 Specific:** Includes FlowModule architecture analysis, multiple connection types, database tables

## Workflow Examples

### Analyzing a Network Game

```bash
# 1. Analyze the ELF structure
/skill:ps2-elf analyze /path/to/online-game.elf

# 2. Extract network-related information
/skill:ps2-elf network /path/to/online-game.elf

# 3. Find relevant strings
/skill:ps2-elf strings /path/to/online-game.elf "server"

# 4. Document the protocol
/skill:ps2-elf document-protocol /path/to/online-game.elf protocol.md

# 5. Create server template
/skill:ps2-elf server-template "Main server - port 7777 - TCP" python
```

### NASCAR 2004 Server Emulation Workflow

```bash
# 1. Analyze the ELF
readelf -h NASCAR.ELF
readelf -S NASCAR.ELF | grep -E 'text|data|rodata'

# 2. Extract network strings
strings NASCAR.ELF | grep -E 'network|connect|recv|send'

# 3. Find connection format
strings NASCAR.ELF | grep 'connecting to'

# 4. Find packet format
strings NASCAR.ELF | grep 'recv:'

# 5. Document the protocol
cd ~/ps2-analysis
cp /path/to/analysis NASCAR.ELF_analysis.md

# 6. Create server template
cd ~/ps2-server
# Use the detected packet format (8 bytes: 4/4) and hex IP format
```

## Output Files

Results are saved to:
- `~/ps2-analysis/` - Analysis outputs
  - `quick_<name>.ELF.txt` - Quick ELF analysis
  - `network_<name>.ELF.txt` - Network protocol extraction
  - `strings_<name>.ELF.txt` - String extraction
  - `strings_<name>.ELF_detailed.txt` - Detailed string analysis
  - `radare2_analysis_<name>.ELF.txt` - Radare2 disassembly
  - `comprehensive_<name>.ELF_analysis.md` - Full protocol documentation
- `~/ps2-protocol/` - Protocol documentation
- `~/ps2-server/` - Server code templates

## PS2 ELF Concepts

### Common Sections
- `.text` - Executable code
- `.data` - Initialized data
- `.bss` - Uninitialized data
- `.rodata` - Read-only data (strings, constants)
- `.rel.dyn` / `.rel.plt` - Relocation tables

### Common Libraries
- `libnet` - PS2 networking library
- `libps2ip` - IP stack for PS2
- `libps2sio` - Serial I/O
- `libpfs` - File system
- `libkernel` - Kernel functions
- `libmc` - Memory Card library

### Common Network Functions
- `connect`, `accept`, `bind`, `listen` - Socket functions
- `send`, `recv`, `sendto`, `recvfrom` - Data transfer
- `inet_addr`, `inet_ntoa` - Address conversion
- `socket`, `close` - Socket creation

### PS2-Specific Patterns
- **FlowModule Architecture:** Middleware pattern for network handling (common in EA games)
- **Connection Types:** UDP, Reliable UDP (guaranteed delivery), Escrow (transactional data)
- **Connection Format:** `connecting to %08x:%d` (hex IP + decimal port)
- **Packet Format:** Often `recv: %c%c%c%c/%c%c%c%c` (8 bytes, 4/4 split)
- **Server Messages:** Maintenance messages, ICMP timeout, RPC errors
- **Database Tables:** PORT, NIPS, KCRT, DIPT, TIPS, TCPG (configuration storage)

## Related Tools

- `radare2` - **Primary tool for MIPS PS2 ELF analysis** (disassembly, strings, analysis)
- `readelf` - ELF information (header, sections)
- `objdump` - Limited MIPS support, use radare2 instead
- `gdb` - Debugging and reverse engineering
- `Wireshark` - Network protocol analysis

## Tips

1. **Start with ELF analysis** to understand the binary structure
2. **Use network extraction** to identify all endpoints
3. **Search strings** to find protocol documentation and constants
4. **Document the protocol** before implementing the server
5. **Start with a simple server** and add features incrementally

### PS2-Specific Tips
- **Use radare2** for advanced disassembly and string analysis
- **Look for FlowModule** patterns in network code
- **Check for multiple connection types** (UDP, Reliable UDP, Escrow)
- **Analyze database tables** for configuration (PORT, NIPS, etc.)
- **Use Wireshark** to capture real traffic for verification
- **Check for hex IP format** in connection strings
- **Look for packet size validation** ("packet size larger than expect")
- **Investigate event observers** for protocol flow

## Advanced Usage

### Custom Disassembly with Radare2

```bash
# List all strings
r2 game.elf -c 'iz'

# Find network-related strings
r2 game.elf -c 'iz~network'

# Disassemble specific function
r2 game.elf -c 'pdf @ 0x401000'

# Export disassembly to file
r2 game.elf -c 'pdf @ 0x401000 > function.txt'

# Find references to a string
r2 game.elf -c 'izz~"connecting to"'
```

### String Pattern Matching

```bash
# Find all strings containing "login"
/skill:ps2-elf strings /path/to/game.elf login

# Find strings with specific patterns
/skill:ps2-elf strings /path/to/game.elf "password"
/skill:ps2-elf strings /path/to/game.elf "packet"

# In radare2
r2 game.elf -c 'izz~"login"'
```

## References

- **NASCAR 2004 Analysis:** `~/ps2-analysis/comprehensive_NASCAR.ELF_analysis.md`
- **PS2 Network Protocols:** Common patterns found in EA Sports titles
- **MIPS R3000 Architecture:** Standard for PS2 games
- **FlowModule:** EA's middleware architecture for network handling
