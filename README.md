# VTSTech's AgentSkills

A curated collection of **AgentSkills** — reusable, modular instruction sets designed to extend and enhance the capabilities of AI coding agents. Each skill is a self-contained, well-documented prompt package that can be loaded into compatible agent platforms to give AI assistants specialized knowledge, workflows, and tooling for specific domains.

## Table of Contents

- [Overview](#overview)
- [What Are AgentSkills?](#what-are-agentskills)
- [Repository Structure](#repository-structure)
- [Included Skills](#included-skills)
  - [ACP — Agent Control Panel](#acp--agent-control-panel)
  - [Codebase Audit — Intelligence Brief & Audit Reports](#codebase-audit--intelligence-brief--audit-reports)
  - [PS2 ELF — Reverse Engineering Toolkit](#ps2-elf--reverse-engineering-toolkit)
- [How to Use These Skills](#how-to-use-these-skills)
- [Skill Conventions](#skill-conventions)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This repository serves as a centralized home for AgentSkills authored by **VTSTech**. Each skill addresses a distinct problem domain — from agent workflow orchestration and codebase analysis to low-level binary reverse engineering — and is packaged as a standalone `SKILL.md` file that can be dropped into any compatible agent framework.

The project is intentionally lightweight. There are no build steps, no runtime dependencies, and no framework-specific lock-ins. Each skill is a Markdown document with structured frontmatter that agents can parse, interpret, and execute against. This makes the collection portable across a wide range of AI agent platforms and orchestrators.

| Detail | Value |
|--------|-------|
| **Repository** | [VTSTech/skills](https://github.com/VTSTech/skills) |
| **License** | MIT |
| **Author** | VTSTech |
| **Created** | April 16, 2026 |

---

## What Are AgentSkills?

AgentSkills are structured prompt documents (in Markdown format) that encode specialized knowledge, step-by-step workflows, API specifications, and decision-making logic for AI agents. Rather than relying solely on an agent's general training data, AgentSkills give agents precise, domain-specific instructions that enable them to perform complex tasks consistently and correctly.

Each skill follows a standardized format:

- **Frontmatter (YAML):** Contains metadata such as the skill name, description, trigger phrases, version, license, and author information. Agents use this metadata to decide when to invoke a skill.
- **Body (Markdown):** Contains the full set of instructions, API references, workflow patterns, examples, and troubleshooting guidance. This is the substantive content that the agent follows during execution.

Think of AgentSkills as **plug-and-play instruction modules** for AI agents. Just as a developer imports a library to gain new capabilities, an AI agent loads a skill to gain new domain expertise and workflow knowledge.

---

## Repository Structure

```
skills/
├── README.md                          # This file
├── LICENSE                            # MIT License
├── .gitignore                         # Git ignore rules
├── acp/                               # ACP — Agent Control Panel skill
│   └── SKILL.md                       #   Full skill definition (v1.0.6)
├── codebase-audit/                    # Codebase Audit skill
│   ├── SKILL.md                       #   Full skill definition (v0.2.0)
│   └── references/                    #   Reference templates
│       ├── brief-template.md          #     Template for intelligence briefs
│       └── audit-template.md          #     Template for audit reports
└── ps2-elf/                           # PS2 ELF Reverse Engineering skill
    └── SKILL.md                       #   Full skill definition (v0.0.1)
```

Each skill resides in its own directory containing at minimum a `SKILL.md` file. Some skills include supplementary reference materials in subdirectories (such as templates and example outputs).

---

## Included Skills

### ACP — Agent Control Panel

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0.6 |
| **Path** | `acp/SKILL.md` |
| **A2A Compliant** | Yes |

The **ACP (Agent Control Panel)** skill is a comprehensive workflow orchestration and session management system for AI agents. It is designed to be invoked **first on every session start**, before any other skill or work begins. ACP provides structured logging, activity tracking, token budget estimation, multi-agent coordination, and human-in-the-loop control — all through a well-defined REST API.

**Core Capabilities:**

- **Bootstrap Protocol:** On session start, the agent performs a 6-step bootstrap sequence (status check, identity verification, agent registration, TODO restoration, initial activity log, and completion) to establish a clean working state. This ensures continuity across sessions and prevents orphaned activities from polluting new work.
- **Mandatory Workflow Pattern (CHECK → LOG → EXECUTE → COMPLETE):** Every action the agent takes must follow this four-phase lifecycle. Before executing any tool or operation, the agent logs the intended action via `POST /api/action`. After execution, it records the result via `POST /api/complete`. This pattern provides full auditability and enables the control panel to detect loops, track progress, and surface orphaned tasks.
- **Activity Tracking with 11 Action Types:** ACP classifies agent actions into typed categories — READ, WRITE, EDIT, BASH, SEARCH, SKILL, API, TODO, CHAT, and A2A — each with its own semantic meaning. This typed system allows the control panel to produce meaningful activity histories and statistics.
- **Stop/Resume Control:** A human operator can set a `stop_flag` via `POST /api/stop` to immediately halt all agent activity. The agent checks this flag before every action and stops processing when it's set. Work can be resumed with `POST /api/resume`, which clears the flag and allows the agent to continue.
- **Nudge System (Human Guidance):** The primary agent (the first agent to log activity in a session) receives human "nudges" — priority-annotated messages from the operator that guide the agent's behavior. Nudges can require acknowledgment (`requires_ack: true`), ensuring the agent has read and processed the guidance before continuing. A dedicated `GET /api/nudge` endpoint allows polling for pending nudges without side effects.
- **Orphan Detection:** If the agent starts new work while previous activities remain incomplete, ACP issues an `orphan_warning` in the API response. The agent must resolve all orphans (by completing or canceling them) before proceeding with new work.
- **Token Tracking & Budget Estimation:** ACP estimates token consumption using a characters-to-tokens ratio (3.5 chars per token). Agents pass `content_size` on actions and completions for accurate tracking. File deduplication ensures re-reading the same file within a session doesn't inflate the token count.
- **Multi-Agent Ownership Model:** In multi-agent sessions, each activity is owned by the agent that created it. Only the owning agent may complete its activities (attempting to complete another agent's task returns HTTP 403). Per-agent token isolation ensures secondary agents don't pollute the primary agent's context window estimate.
- **A2A (Agent-to-Agent) Messaging:** Agents can send typed messages (request, response, notification) to each other with priority levels, TTL, and reply chaining. This enables complex multi-agent workflows where agents coordinate on shared tasks.
- **TODO Synchronization:** ACP provides endpoints for creating, updating, toggling, and clearing TODO items. All TODO state changes are logged as activities, providing a full history of task state transitions.
- **File Manager API:** A full file management API with endpoints for listing, viewing, uploading, downloading, saving, deleting, creating directories, extracting archives (zip, tar, gz, bz2), and compressing files.
- **Context Recovery:** Before context compression or session end, agents can save notes (categorized as decisions, insights, context, warnings, or todos) and export a session summary to a persistent Markdown file. On session resume, the agent reads this summary to quickly restore its working state.
- **Graceful Shutdown:** `POST /api/shutdown` triggers a graceful session teardown — it exports the session summary, cancels running activities, delivers a shutdown nudge to the primary agent, and stops the server after a 2-second cooldown period.

**Triggers:** Invoke on every session start, context resume, or context reset. Required before any other work.

---

### Codebase Audit — Intelligence Brief & Audit Reports

| Attribute | Value |
|-----------|-------|
| **Version** | 0.2.0 |
| **Path** | `codebase-audit/SKILL.md` |
| **References** | `references/brief-template.md`, `references/audit-template.md` |

The **Codebase Audit** skill solves a critical efficiency problem: understanding an unfamiliar codebase typically consumes most of an agent's context window budget before any productive work even begins. This skill provides a structured workflow for rapidly analyzing any codebase and producing two complementary output documents — a concise **intelligence brief** (`brief.md`) and a detailed **audit report** (`audit.md`) — that allow subsequent agents or sessions to skip the discovery phase entirely and begin execution with their full context window available.

**Core Capabilities:**

- **Two-Mode Operation:** The skill operates in two modes depending on whether a brief already exists. In **Load Mode** (brief found), the agent reads the existing brief for instant orientation and proceeds directly to the user's task. In **Audit Mode** (brief missing), the agent performs a full codebase analysis and generates both the brief and the audit report from scratch.
- **Efficient Discovery Strategy:** Rather than reading every file, the skill guides the agent through a prioritized scanning sequence: package/lock files (tech stack identification), config files (build tools and frameworks), directory structure (project organization), README/docs (if available), entry points (main files and route definitions), and type definitions/schemas. The agent then selectively reads files in dependency chains from entry points, skipping generated files, build artifacts, and vendor directories entirely.
- **Intelligence Brief (brief.md):** A condensed orientation document designed to be a "map, not a diary." The brief answers the question "What does the next agent need to know to be effective?" It is token-budgeted at a maximum of 16,000 tokens (approximately 56,000 characters) to ensure compatibility with models ranging from 32K to 256K context windows. The brief uses the provided template (`references/brief-template.md`) for consistent structure across audits.
- **Audit Report (audit.md):** A comprehensive findings document organized into seven severity categories with unique ID prefixes:
  - **SEC** — Security vulnerabilities, input validation, injection risks, data exposure
  - **ROB** — Robustness issues, error handling, retry logic, graceful degradation
  - **MAINT** — Maintainability concerns, complexity, code duplication, developer experience
  - **PERF** — Performance optimizations, caching, streaming, resource management
  - **FEAT** — New feature suggestions grounded in observed gaps or friction points
  - **ARCH** — Architecture patterns, inter-module communication, extensibility
  - **TEST** — Testing gaps, missing test types, CI/CD improvements

  Each finding includes a severity rating, affected files, concrete code references, and a one-sentence impact statement. The report also features a **Priority Matrix** (timeline-based) and an **Architecture Strengths** section that documents what the codebase does well to prevent good patterns from being lost during refactoring.

- **Content Quality Standards:** Every finding must reference specific files and line sections (no vague claims), severity ratings must be grounded in actual impact, and new feature suggestions must be based on observed gaps rather than speculative ideas. Categories with no findings are omitted from the detailed section.
- **Optional Formatted Deliverables:** If the user requests a PDF or DOCX, the audit.md content can be fed into document generation skills to produce a polished, styled report. The brief and audit.md are always generated as Markdown regardless.

**Triggers:** "Audit this repo", "review the codebase", "what does this project do", "get me up to speed", "brief me on", "understand this code", "code review", or when starting work on an unfamiliar codebase.

---

### PS2 ELF — Reverse Engineering Toolkit

| Attribute | Value |
|-----------|-------|
| **Version** | 0.0.1 |
| **Path** | `ps2-elf/SKILL.md` |

The **PS2 ELF** skill is a specialized reverse engineering toolkit for analyzing PlayStation 2 game executables, extracting network protocols, and building server emulators. It provides structured workflows for working with MIPS R3000 architecture ELF binaries and integrates with industry-standard tools like radare2 for disassembly and analysis. The skill includes a real-world case study based on the analysis of NASCAR 2004 EA SPORTS.

**Core Capabilities:**

- **ELF Analysis:** Extracts and reports on ELF header information, section properties, entry points, code sections, string tables, symbol names, and commonly used PS2 libraries (libnet, libps2ip, libps2sio, libpfs, libkernel, libmc). Detects MIPS R3000 architecture specifics including little-endian byte order and System V ABI.
- **Network Protocol Extraction:** Scans ELF binaries for network-related functions (send, recv, connect, accept, bind, listen), extracts known IP addresses and port numbers, identifies protocol constants and packet sizes, lists potential server endpoints, and finds string literals related to networking. PS2-specific detection includes FlowModule architecture patterns (common in EA games), multiple connection types (UDP, Reliable UDP, Escrow), and database configuration tables.
- **String Analysis:** Extracts all printable strings from ELF binaries with optional pattern filtering. Groups strings by context and identifies protocol strings, error messages, server messages, and configuration tables. Includes radare2 integration for advanced string searching.
- **Disassembly Export:** Uses radare2 (preferred over objdump for MIPS) to disassemble specified sections or address ranges, with output directed to files for further analysis. Supports targeted function disassembly and cross-reference analysis.
- **Server Template Generation:** Creates basic server skeleton code in Python or Go based on extracted protocol information. Templates include hex IP parsing, multi-byte packet handling, and connection timeout management — all tailored to PS2 game server patterns.
- **Protocol Documentation:** Produces comprehensive protocol documentation covering all detected endpoints, packet structures, example packet formats, and suggested server implementation approaches. Includes FlowModule architecture analysis where applicable.

**Case Study — NASCAR 2004 EA SPORTS:**
The skill includes a detailed analysis of NASCAR.ELF (3.1 MB, MIPS R3000, stripped), documenting:
- Entry point at `0x100008`
- 8-byte packet format (`recv: %c%c%c%c/%c%c%c%c`)
- Connection string format (`connecting to %08x:%d` — hex IP with decimal port)
- Multiple connection types: UDP, Reliable UDP, Escrow, and FlowModule-based
- EA's FlowModule middleware architecture pattern
- Server maintenance messages and ICMP timeout handling
- Configuration database tables (PORT, NIPS, KCRT, DIPT, TIPS, TCPG)

**Output Structure:** Analysis results are organized into `~/ps2-analysis/` (raw analysis outputs and comprehensive documentation), `~/ps2-protocol/` (protocol documentation), and `~/ps2-server/` (server code templates).

**Triggers:** Use when analyzing PS2 game executables, extracting network protocols from PS2 binaries, building PS2 game server emulators, or performing MIPS R3000 reverse engineering.

---

## How to Use These Skills

1. **Clone the repository:**
   ```bash
   git clone https://github.com/VTSTech/skills.git
   ```

2. **Load a skill into your agent platform:**
   Copy the `SKILL.md` file from the desired skill directory into your agent's skill/loadable directory. Consult your agent platform's documentation for the specific location and format requirements.

3. **Invoke skills via trigger phrases:**
   Each skill's frontmatter includes a `description` field with natural language trigger phrases. When your agent encounters one of these phrases in a user prompt, it should automatically load and follow the corresponding skill's instructions.

4. **Skill loading order matters:**
   The **ACP** skill is explicitly designed to be loaded **first** on every session start. It provides session management and workflow orchestration that other skills rely on. Always invoke ACP before any other skill.

---

## Skill Conventions

All skills in this repository follow a consistent set of conventions:

- **Standardized Frontmatter:** Every `SKILL.md` begins with YAML frontmatter containing `name`, `description`, `license`, and `metadata` (author, version, repository). The `description` field includes trigger phrases for automatic skill selection.
- **Self-Contained:** Each skill is fully self-contained in its directory. There are no cross-skill dependencies at the file level, though the ACP skill is intended to be invoked first for workflow management purposes.
- **Markdown-Based:** All skills are written in Markdown for maximum portability. They can be read by any text editor, rendered by any Markdown processor, and parsed programmatically.
- **MIT Licensed:** All skills are released under the MIT License, permitting free use, modification, and distribution.
- **Versioned:** Each skill tracks its own version independently using semantic versioning (or pre-release versioning for early-stage skills).
- **Reference Materials:** Skills that benefit from supplementary templates or reference documents include them in a `references/` subdirectory.

---

## Contributing

Contributions are welcome. If you'd like to add a new skill or improve an existing one:

1. Fork the repository
2. Create a new directory for your skill (or modify an existing one)
3. Ensure your `SKILL.md` follows the frontmatter conventions described above
4. Test your skill with a compatible agent platform
5. Submit a pull request with a clear description of the skill's purpose and capabilities

---

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for the full text.
