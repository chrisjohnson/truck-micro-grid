# AGENTS.md — opencode Project Context

This file tells opencode how to interact with this project: a custom 12V DC micro-grid for a 2024 Ford F250 Tremor with a SmartCap camper shell.

The master spec has been split into focused files under `spec/` for optimal agentic loading. Always load only the spec files needed for the task at hand.

## Project Files

| File | Purpose |
|------|---------|
| `spec/01-system-core.md` | **Planner's primary load.** Topology (S3), Physics (S6), Fuse Audit (S10), Energy Balance (S11), Tech Debt (S12) |
| `spec/02-wiring-schedule.md` | Point-to-point wiring & connection table (S9). Load with planner/coder for circuit changes. |
| `spec/03-controls-settings.md` | Control logic, relay truth table, VictronConnect programming (S4, S13). Load for configuration changes. |
| `spec/04-hardware-layout.md` | Hardware inventory, bus bar distribution, battery box internals (S2, S5, S5A). Load for component adds. |
| `spec/05-installation-safety.md` | System overview, parasitic strategy, mechanical install, solar safety (S1, S7, S8). Load for safety reference. |
| `master_design_spec.md` | Index / file map. Lightweight entry point. |
| `AGENTS.md` | This file — opencode agent instructions |
| `opencode.json` | Agent/plugin configuration |

## System Identity

- **Platform:** 2024 Ford F250 Lariat Tremor (7.3L Gas, SmartCap)
- **Architecture:** Dual-battery solar-assisted 12V DC micro-grid
- **House Bank:** 2× Goldenmate 100Ah LiFePO4 (removable battery boxes)
- **Starter:** OEM lead-acid (maintained by solar)
- **Solar:** 1× Renogy 200W Shadowflux N-Type, Victron SmartSolar MPPT 100/20
- **Charging:** Victron Orion-Tr Smart 12/12-18A (alternator), Cyrix-Li-ct combiner (solar bridge)
- **Key Design Choice:** Dedicated negative return highway (no chassis ground), SW6 always-on with solar offset

## How opencode Agents Should Approach This System

### General Principles

- **Always consult the spec files** before suggesting wire gauge changes, fuse changes, topology changes, or Victron programming changes. The spec contains exactly-once documented physics calculations and trade-offs.
- **Do not suggest generic "best practices" that contradict the spec's deliberate compromises** (e.g., CCA wire debt, XT60 standardization, ground shift acceptance). These are accepted technical debt with documented mitigation (`spec/01-system-core.md` Section 12).
- **When troubleshooting, trace the signal path.** The system has a strict hierarchy: SW6 → Frame Rail T-Junction → Anderson → Upper Board Bus → (Orion|Cyrix) → Lower Board Bus → Battery Boxes → Loads. Follow the topology diagram (`spec/01-system-core.md` Section 3).
- **The user is the domain expert.** This is a real deployed physical system. Do not second-guess verified engineering. Ask clarifying questions when the spec is ambiguous.
- **Safety first.** This is a 200W solar + 200Ah LiFePO4 + 100Ah lead-acid setup. Always consider:
  - Arc-flash risk (PV is always live in daylight, `spec/05-installation-safety.md` Section 8)
  - CCA termination protocols (`spec/05-installation-safety.md` Section 7 — Noalox, ferrules, hydraulic crimp)
  - Battery box commissioning rules (100% SoC before parallel, `spec/05-installation-safety.md` Section 7)
  - Fuse coordination (`spec/01-system-core.md` Section 10 — which fuse blows first)
  - The Golden Safety Sequence for solar maintenance (`spec/05-installation-safety.md` Section 8)

### Agent Routing Guide (for orchestrator)

You are the **orchestrator** (Gemma 4 Edge). Your job is routing. Follow these rules:

| If the user asks about... | Route to... | Because... |
|---|---|---|
| Circuit changes, new wire runs, topology, voltage drop math, fuse sizing, component upgrades, energy balance, failure mode analysis | `planner` | Heavy systems engineering with physics requirements |
| Editing the spec (adding/removing sections, updating tables, fixing formatting), updating wiring schedules, fuse audits, Victron settings tables | `coder` | Precision markdown editing |
| Troubleshooting a specific symptom, "what does this section say?", navigating the spec, quick lookups | Handle yourself (orchestrator) | Simple Q&A needs no subagent |

If uncertain, route to `planner` — better to over-scope to engineering depth.

### How Each Subagent Should Operate

#### `planner` — Electrical system design
When invoked:
1. Read `spec/01-system-core.md` Sections 3, 6, 10, and 12 first.
2. Perform voltage drop and energy balance math to verify proposals.
3. Check fuse coordination (`spec/01-system-core.md` Section 10) — ensure new circuit doesn't create nuisance trip risk.
4. Propose the change with before/after physics comparison.
5. Write the spec update as a task description for coder to execute.

Useful spec references per task:
- **New wire or component:** `spec/02-wiring-schedule.md` (Section 9), `spec/04-hardware-layout.md` (Section 5 bus bars, Section 2 inventory)
- **Programming/configuration:** `spec/03-controls-settings.md` (Section 13 VictronConnect settings)
- **Physics/voltage drop:** `spec/01-system-core.md` Section 6 (impedance verification)
- **Fuse question:** `spec/01-system-core.md` Section 10 (fuse audit)
- **Energy balance:** `spec/01-system-core.md` Section 11 (parasitic modeling, solar offset)
- **Known issues:** `spec/01-system-core.md` Section 12 (technical debt)

#### `coder` — File modifications
When editing spec files:
- Preserve the section numbering scheme
- Keep Mermaid diagrams consistent with any text changes
- Update the wiring schedule (`spec/02-wiring-schedule.md` Section 9) and fuse audit (`spec/01-system-core.md` Section 10) in lockstep when modifying circuits
- Update the energy balance (`spec/01-system-core.md` Section 11) when changing loads, solar, or parasitic draws
- Update Victron settings (`spec/03-controls-settings.md` Section 13) when changing hardware
- Do NOT make assumptions about wire gauges or fuses — these are physics-determined, not guessed

### For Any Agent Doing Quick Spec Lookups

When you need to navigate the spec without a full read:
- `Section X` — jump to a section number
- Component names: `Orion`, `Cyrix`, `SmartShunt`, `BatteryProtect`, `MPPT`
- Wire identifiers: `8 AWG CCA`, `House Charging Highway`, `Ignition Signal Line`
- Fuse identifiers: `40A JCase`, `30A MIDI`, `50A MRBF`
- Settings: `Lock-out Threshold`, `Absorption Voltage`, `Remote On/Off`

## Design Principles (Do Not Violate)

1. **No chassis ground.** The dedicated negative return highway is inviolable. All negatives return to the starter battery (-) lug, not the frame.
2. **SW6 is always on.** The solar offset strategy (`spec/01-system-core.md` Section 11) depends on this. Do not suggest toggling SW6 off as a normal operating procedure.
3. **The Cyrix is the automatic load-shedding device.** When starter voltage drops below 12.8V, the Cyrix opens, isolating house loads. This is the primary safety mechanism.
4. **The 5-pin SPDT relay interlocks Orion and Cyrix.** They are never both active. Engine running = Orion active, Cyrix locked open. Engine off = Cyrix armed, Orion forced off.
5. **PV must be physically disconnected** (roof MC4) before any service. The MPPT solar disable switch does NOT isolate PV conductors (`spec/05-installation-safety.md` Section 8).
6. **100% SoC balance before parallel battery connection.** This is non-negotiable to prevent fuse racing (`spec/05-installation-safety.md` Section 7, `spec/01-system-core.md` Section 10).

## Questions to Ask the User

When the context requires decisions, ask:

### Design & Planning
- *"Do you want to change the architecture or just optimize the existing one?"*
- *"Is this a new load to add, or a modification to an existing circuit?"*
- *"Have you verified the current draw of the new component?"*
- *"Where will this new component be physically located (engine bay, bed, tailgate, roof)?"*

### Troubleshooting
- *"What symptoms are you seeing? (voltage readings, fuse blows, unexpected shutdowns, Bluetooth warnings)"*
- *"Are both house batteries connected, or just one?"*
- *"What state is the ignition in when this happens?"*
- *"What's the solar disable switch set to?"*
- *"Is this happening while driving or parked?"*
- *"Have you checked the VictronConnect app for error codes on the Orion, MPPT, or SmartShunts?"*

### Installation
- *"Have you confirmed the battery boxes are both at 100% SoC before parallel connection?"*
- *"Are you using CCA termination protocols (Noalox, ferrules, hydraulic crimp)?"*
- *"Which fuse are you landing this on — charging bus bar or load bus bar?"*

### Failure Modes
- *"Are you seeing nuisance fuse trips? On which fuse?"* (Reference `spec/01-system-core.md` Section 10 for known risks)
- *"Is the Cyrix chattering?"* (Should not happen due to relay interlock, but could indicate a control wiring fault)
- *"Is the Orion shutting down unexpectedly?"* (Check `spec/03-controls-settings.md` Section 13 lock-out thresholds)

## Common Workflows

### Adding a new 12V load
1. Determine if it connects to Charging Bus Bar (unprotected) or Load Bus Bar (protected via BatteryProtect)
2. Check `spec/04-hardware-layout.md` Section 5 for available bus bar studs
3. If full, either replace an existing load or add a new bus bar
4. Check `spec/01-system-core.md` Section 6 for voltage drop on the wire run
5. Size fuse per `spec/01-system-core.md` Section 10 guidelines
6. Update `spec/02-wiring-schedule.md` (Section 9) and `spec/01-system-core.md` Section 10 fuse audit

### Troubleshooting a blown fuse
1. Identify which fuse from `spec/01-system-core.md` Section 10
2. Trace the protected circuit in `spec/02-wiring-schedule.md` Section 9
3. Check for short circuits, overcurrent, or inrush
4. For inrush-sensitive circuits (Cyrix 30A), reference the soft start procedure
5. For XT60 patch cable issues, verify battery balance (`spec/05-installation-safety.md` Section 7)

### Commissioning (first power-on)
1. Verify all fuses are correct values and installed
2. Confirm batteries at 100% SoC before connecting Charge ports
3. Apply VictronConnect settings per `spec/03-controls-settings.md` Section 13
4. Test relay logic: ignition off → Cyrix active, Orion off; ignition on → Orion active, Cyrix locked
5. Verify solar disable switch controls MPPT output

### Planning a component upgrade
1. Formulate as a planner task with the specific change
2. Reference the relevant sections for existing design
3. Run voltage drop and energy balance math per `spec/01-system-core.md` Section 6 and 11 style
4. Check fuse coordination per `spec/01-system-core.md` Section 10 methodology
5. Document the change by editing the relevant spec file(s)

## Failure Modes to Monitor

| Failure | Symptom | Root Cause | Action |
|---------|---------|------------|--------|
| Nuisance 30A Cyrix fuse blow | House bank not charging via solar | Deeply discharged house batteries + Cyrix bridge inrush | Use soft start: SW6 off → start engine → wait for Orion → SW6 on |
| Nuisance 40A JCase engine bay | Orion + tailgate total load near limit | Heat-soaking in summer + combined draw ~35A | Monitor; plan upgrade to 8 AWG OFC if persistent (`spec/01-system-core.md` Section 12 debt) |
| Orion premature shutdown | Engine off but Orion still running briefly | Input voltage lock-out too low | Check `spec/03-controls-settings.md` Section 13 settings (Lock-out: 12.6–12.7V) |
| Ground-shift false readings | SmartShunt shows odd voltages while driving | Accepted non-isolated DC-DC common-mode shift | Apply `spec/03-controls-settings.md` Section 13 calibration offsets; check Cyrix is immobilized by relay |
| Battery box doesn't charge | No current through charge patch | BMS LVD latched (drained below 11.0V) | Jump-start battery box via XT60 load port temporarily to wake BMS |
| SmartCap bed lights always-on | LED indicator on local switch | By design — SW6 always hot | Normal; verify draw is included in 25mA parasitic budget (`spec/01-system-core.md` Section 11) |

## System Physics Reference

Use these for quick calculations (detailed in `spec/01-system-core.md` Section 6):

- **Main highway loop resistance:** ~0.08Ω (50ft 8 AWG CCA frame run + 15ft vertical + connections)
- **Max uncontrolled Cyrix bridge current:** Limited by Ohm's law — 30A draw needs house bank at ~11.0V
- **Fridge circuit drop:** 16ft loop 12 AWG copper → 0.026Ω → 0.13V drop at 5A surge
- **Parasitic baseline (solenoid open):** 255mA → 6.12 Ah/day
- **Solar de-rated output:** ~5.12A × 3.5–5.0 PSH = 18–26 Ah/day (OH driveway)
- **Net surplus (OH home, spring-fall):** ~14 Ah/day positive
