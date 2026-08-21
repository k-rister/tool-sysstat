# Sysstat Tool

## Purpose
Collects system performance metrics using Linux sysstat utilities (mpstat, sar, iostat, pidstat) during benchmark execution and post-processes the raw output into crucible's canonical metric format.

## Languages
- Bash: collection scripts (`sysstat-start`, `sysstat-stop`)
- Python: post-processor (`sysstat-post-process.py`)

## Key Files
| File | Purpose |
|------|---------|
| `sysstat-start` | Launches configured subtools with `--subtools` and `--interval` parameters |
| `sysstat-stop` | Kills running collectors, compresses output with xz |
| `sysstat-post-process.py` | Parses raw sysstat output into crucible metrics using CDMMetrics |
| `rickshaw.json` | Rickshaw integration: endpoint allow/block lists, file deployment, post-process script |
| `workshop.json` | Engine image build: compiles sysstat v12.5.1 from source |
| `tool-metadata.json` | Machine-readable description, subtool list, and CDM-indexed status (consumed by `crucible tools list`) |
| `multiplex.json` | Parameter validation rules and `defaults` preset for multiplex (mirrors benchmark `multiplex.json`) |

## Configuration
- `--subtools <list>` — Comma-separated subtools to run (default: `mpstat,sar,iostat,pidstat`)
- `--interval <seconds>` — Collection interval (default: `3`)

## mpstat CPU breakout dimensions
Beyond `num` (CPU number), mpstat metrics carry these CDM breakout dimensions, all derived from sysfs CPU topology data collected by `sysstat-start` and resolved via toolbox's `system_cpu_topology.build_cpu_topology()`/`get_cpu_topology()`/`get_cpu_node()`/`get_cpu_cache_domains()`:
- `cpu` — string alias for `num`, matching the naming already used by `tool-procstat`/`tool-kernel`
- `node` — NUMA node, when determinable. Collected into a `cpu-numa-nodes.txt` manifest by `sysstat-start` rather than copied directly from sysfs: a CPU's `nodeN` entry is a symlink, and sysfs symlinks report a zero apparent size, which breaks `cpio`'s `readlink()` call
- `shared-lN-domain` (e.g. `shared-l1-domain`, `shared-l3-domain`) — one per cache level present on the host, valued as the formatted CPU range sharing that cache instance (e.g. `"0-15"`). Dynamic per-platform on the collection side; registered in CDM's schema as explicit `shared-l1-domain` through `shared-l4-domain` fields rather than a wildcard match

## Conventions
- Primary branch is `master`
- Runs as a profiler tool on master/worker/profiler roles, blocked on client/server
- Standard Bash modelines and 4-space indentation
