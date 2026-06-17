# Mobile Electrical & Solar Micro-Grid: Master Design & Verification Spec

**System Owner:** Christopher Johnson  
**Platform:** 2024 Ford F250 Lariat Tremor (7.3L Gas, SmartCap Bed Camper Shell)

---

This file is the **index** for the split spec. Detailed content has been organized into five focused files under `spec/` for faster agent loading. Subagents should load only the files needed for their task.

## File Map

| File | Contents | Primary Consumer |
|------|----------|-----------------|
| [`spec/01-system-core.md`](spec/01-system-core.md) | Topology (S3), Physics (S6), Fuse Audit (S10), Energy Balance (S11), Tech Debt (S12) | **Planner** — always load for any circuit/component change |
| [`spec/02-wiring-schedule.md`](spec/02-wiring-schedule.md) | Point-to-point wiring & connection table (S9) | **Planner + Coder** — load when modifying circuits |
| [`spec/03-controls-settings.md`](spec/03-controls-settings.md) | Control logic, relay truth table, VictronConnect settings (S4, S13) | **Coder** — load when changing programming/hardware |
| [`spec/04-hardware-layout.md`](spec/04-hardware-layout.md) | Hardware inventory, bus bar distribution, battery box internals (S2, S5, S5A) | **Planner** — load when adding components or checking bus capacity |
| [`spec/05-installation-safety.md`](spec/05-installation-safety.md) | System overview, parasitic strategy, mechanical installation, solar safety (S1, S7, S8) | **Orchestrator** — occasional reference; safety protocols |

## Quick Reference — Section Origins

| Original Section | Moved To |
|-----------------|----------|
| 1. System Overview & Parasitic Drain | `spec/05-installation-safety.md` |
| 2. Core Hardware Inventory | `spec/04-hardware-layout.md` |
| 3. Physical Layout & Wiring Topology | `spec/01-system-core.md` |
| 4. Control Logic & Switch Configuration | `spec/03-controls-settings.md` |
| 5. Main House Panel Distribution | `spec/04-hardware-layout.md` |
| 5A. Battery Box Internals | `spec/04-hardware-layout.md` |
| 6. System Physics & Impedance | `spec/01-system-core.md` |
| 7. Mechanical Installation & Mounting | `spec/05-installation-safety.md` |
| 8. Solar PV Safety Protocol | `spec/05-installation-safety.md` |
| 9. Wiring & Connection Schedule | `spec/02-wiring-schedule.md` |
| 10. Fuse Audit & Coordination | `spec/01-system-core.md` |
| 11. Energy Balance & Ecosystem Physics | `spec/01-system-core.md` |
| 12. Accepted Technical Debt | `spec/01-system-core.md` |
| 13. VictronConnect Settings | `spec/03-controls-settings.md` |

## System Identity

- **Architecture:** Dual-battery solar-assisted 12V DC micro-grid
- **House Bank:** 2× Goldenmate 100Ah LiFePO4 (removable battery boxes)
- **Starter:** OEM lead-acid (maintained by solar)
- **Solar:** 1× Renogy 200W Shadowflux N-Type, Victron SmartSolar MPPT 100/20
- **Charging:** Victron Orion-Tr Smart 12/12-18A (alternator), Cyrix-Li-ct combiner (solar bridge)
- **Key Design Choice:** Dedicated negative return highway (no chassis ground), SW6 always-on with solar offset
