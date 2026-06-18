# Paranoya

This will be a Linux distribution that is aware of all changes that are happening in the system, for example: when you start an application or service, what directories/files it accessed/wrote, what ports were opened, inbound/outbound network traffic, addresses, how much resources it consumed, what hardware it accessed, system calls, unusual CPU/GPU/TPU instructions, etc. The whole app profile will be stored in a special protected/encrypted place. This distribution is great to test new/suspicious apps, learn how Linux OS works, entertain cyber-security specialists and people with paranoia.

Ideally it should be able to profile not only apps, but users, daemons, AIs, ghosts, vampires and aliens.

## Translation to corporate language (copilot generated):

Paranoya is a Linux distribution concept focused on **full-system behavioral observability** for applications and services.

It is designed for:
- security research,
- malware/suspicious software analysis,
- Linux internals learning,
- "trust but verify" users who want deep runtime visibility.

The objective is to make every meaningful runtime action inspectable and attributable: process creation, file access, network traffic, hardware access, resource usage, and unusual execution patterns.

---

## Core Idea

When an application starts, Paranoya should continuously profile and record:

- **Execution**: process tree, parent/child relationships, command lines, loaded libraries.
- **Filesystem activity**: files/directories read, written, created, deleted, permissions changed.
- **Network behavior**: ports opened, protocol usage, inbound/outbound flows, destination addresses/domains.
- **IPC and service interactions**: DBus activity, socket use, signals, shared memory, service dependencies.
- **System calls and kernel-level events**: syscall frequency, sensitive syscall sequences, privilege transitions.
- **Resource usage**: CPU, RAM, disk IO, GPU/TPU usage, cgroup/container limits.
- **Hardware access**: camera, microphone, USB, Bluetooth, block devices, PCI endpoints.
- **Potentially suspicious behavior**: persistence attempts, tampering with system binaries, anti-analysis patterns.

All profiles are stored in a protected and encrypted telemetry vault.

---

## High-Level Architecture

```text
+-----------------------------+
| Analyst UI / CLI / API      |
+-------------+---------------+
              |
              v
+-----------------------------+
| Correlation + Detection     |
| (rules, anomaly scoring)    |
+-------------+---------------+
              |
              v
+-----------------------------+
| Event Pipeline              |
| normalize -> enrich -> tag  |
+-------------+---------------+
              |
              v
+-----------------------------+
| Collectors                  |
| eBPF | auditd | netflow     |
| perf | procfs | journald    |
+-------------+---------------+
              |
              v
+-----------------------------+
| Secure Telemetry Vault      |
| encrypted, signed, append   |
+-----------------------------+
```

### Architectural Layers

1. **Collection Layer**
   - Kernel/user-space collectors capture signals with minimal overhead.
   - Prefer eBPF-based instrumentation for deep yet efficient visibility.

2. **Normalization Layer**
   - Convert raw events into a common schema (`timestamp`, `host`, `process`, `action`, `target`, `context`).
   - Deduplicate noisy events and preserve ordering.

3. **Correlation Layer**
   - Join file/network/syscall/process events into a single per-app timeline.
   - Build behavior graphs and baseline normal activity.

4. **Detection Layer**
   - Rule-based detections (known bad patterns).
   - Statistical/anomaly detections for unusual behavior.

5. **Storage Layer (Vault)**
   - Encrypted at rest.
   - Integrity protected (hash chaining / signatures).
   - Optional WORM/append-only mode for forensics.

6. **Presentation Layer**
   - CLI + web UI + export APIs.
   - Query by app, timeframe, IOC, destination, syscall, etc.

---

## Suggested Software Stack

A practical open-source stack for a first implementation:

### OS/Foundation
- **Base distro**: Debian stable or Fedora (strong packaging + security updates).
- **Kernel**: recent LTS kernel with eBPF features enabled.
- **Init/service manager**: systemd.

### Telemetry Collection
- **eBPF**: bpftrace / libbpf-based probes (process, syscall, network, file events).
- **audit framework**: `auditd` for policy-driven auditing and compliance-style logs.
- **Network visibility**: nftables logging, conntrack events, optional Zeek/Suricata sensors.
- **Performance/resource**: `perf`, cgroup metrics, PSI (pressure stall information), GPU telemetry tools.

### Pipeline & Processing
- **Message bus**: NATS / Kafka / Redpanda (start with NATS for simplicity).
- **Stream processing**: Vector / Fluent Bit + custom enrichment service.
- **Detection engine**: Sigma-like rules + custom behavioral heuristics.

### Storage
- **Hot store (searchable)**: OpenSearch or ClickHouse.
- **Time-series metrics**: Prometheus.
- **Long-term archive**: encrypted object storage or append-only filesystem snapshots.

### Security Controls
- **Encryption**: LUKS/dm-crypt for disk + age/sops or KMS-backed key wrapping for exports.
- **Integrity**: event signing (e.g., Ed25519), tamper-evident hash chains.
- **Access control**: RBAC + MFA for analyst interfaces.

### UI & Developer Experience
- **API**: gRPC/REST service for querying events and profiles.
- **UI**: lightweight web frontend with timeline/graph views.
- **CLI**: `paranoya` tool for profile diff, trace replay, and IOC search.

---

## Data Model (Minimal)

Each event should map to a shared structure:

- `event_id`
- `timestamp`
- `host_id`
- `session_id` (analysis session/sandbox run)
- `process` (pid, ppid, uid, gid, exe, cmdline, container info)
- `event_type` (`file_open`, `net_connect`, `syscall`, `usb_access`, ...)
- `target` (file path, IP:port, device, syscall name)
- `result` (success/failure, errno)
- `risk_tags` (optional)
- `raw_payload` (optional original data)

This makes cross-source correlation easier and keeps the architecture extensible.

---

## Example Workflow

1. Analyst launches suspicious binary in isolated profile/sandbox.
2. Collectors capture all runtime behavior.
3. Pipeline normalizes and correlates events.
4. Detection engine flags suspicious patterns (e.g., mass file encryption + external C2 connection).
5. Analyst inspects generated app profile timeline.
6. Profile and evidence are exported with integrity metadata.

---

## Security & Privacy Considerations

- Store telemetry in encrypted partitions by default.
- Minimize data leakage in exports (redaction policies).
- Ensure strict separation between host and analysis workloads.
- Sign critical artifacts and keep verification tooling simple.
- Document legal/ethical constraints for network inspection and malware handling.

---

## Roadmap (Suggested)

### Phase 1: MVP
- Process/file/network event capture.
- Unified event schema.
- Local encrypted storage.
- Basic CLI queries.

### Phase 2: Correlation & Rules
- Cross-source timeline view.
- Detection rules for suspicious behavior.
- Per-application profile generation.

### Phase 3: Advanced Analysis
- Behavioral baselining and anomaly scoring.
- Hardware access profiling (GPU/TPU/USB).
- Remote collectors and distributed analysis nodes.

---

## Project Status

This repository currently contains the project vision and architecture proposal. Implementation can proceed incrementally using the roadmap above.
