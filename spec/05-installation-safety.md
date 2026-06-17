# System Overview, Installation & Safety Protocols

**Cross-reference:** This file combines Sections 1, 7, and 8 of the master spec. For the energy balance math that justifies the SW6 always-on strategy, see `spec/01-system-core.md` (Section 11).

---

## 1. System Overview & Parasitic Drain Analysis

This system is a dual-battery, solar-assisted 12V DC micro-grid spanning the engine bay, the truck frame rails (for routing only), and the truck bed. It utilizes a **dedicated negative return highway** directly to the starter battery to ensure complete electrical isolation from the truck's chassis.

### ⚠️ Parasitic Drain & Solar Offset Strategy
In the current setup, the main 8 AWG CCA power highway to the tailgate T-junction is powered from the output of **Upfitter Switch 6 (SW6)**, which is configured in the engine compartment to be "hot-at-all-times" (always hot).
* **The Draw (Upfitter Relay Coil):** Leaving the cab's SW6 switch toggled ON keeps the factory upfitter relay coil energized continuously. This relay coil draws **$\sim150\text{–}200\text{mA}$** of holding current.
* **The Strategy (Solar Offset):** Rather than toggling SW6 off, the design strategy is to **leave SW6 toggled ON 24/7** so that the tailgate bus bar and custom bed lights are always active. The continuous draw of the upfitter relay coil ($\sim150\text{–}200\text{mA}$), the Cyrix combiner solenoid when bridged ($\sim220\text{–}300\text{mA}$), and the F250's factory idle draw are intended to be fully offset by the daily energy harvest of the 200W solar panel (which prioritizes keeping the starter battery topped up).
* **Custom vs. OEM Lights:** The custom SmartCap LED light strips (referred to as **truck bed lights**) run off the tailgate fuse panel. These custom lights are controlled by a local switch with an always-on LED, which is distinct from the F250's OEM factory bed lights that feature an automatic timeout. Both systems draw power from the truck's single starter battery.

---

## 7. Mechanical Installation, Mounting & Thermal Mitigation Standards

### 🔩 Board Mounting & Structural Layout
* **Upper Sub-Board (SmartCap Ceiling):** Bolted securely to the SmartCap's integrated ceiling **M8 threaded inserts** using lock washers and fender washers to resist vibration during off-roading.
* **Orion Horizontal Thermal Sandbox:** The Victron Orion-Tr Smart is horizontally flush-mounted to the ceiling panel plywood with its aluminum heat-dissipation fins facing up toward the wood backing. To resolve this structural thermal trap, a **1/4" or 3/8" thick aluminum heat spreader plate** is sandwiched directly between the Orion cooling fins and the plywood board surface. 
  * *Extended Footprint Radiator Matrix:* The aluminum plate is cut oversized relative to the Orion casing, extending **2 to 3 inches beyond the top and bottom bounds** of the charger body to form lateral conductive cooling wings.
  * *Dynamic Forced Convection:* When driving, the vehicle's motion activates the SmartCap's integrated roof positive pressure vent, driving massive outside air volume into the bed. This turbulent air stream passes directly over the exposed aluminum spreader wings, continuously pulling away heat drawn conductively through the plate from the fin channels.
* **Lower Sub-Board (Tub Lip Wall):** Mounted using **4x 66lb rubberized neo-magnets** (providing a total of 264 lbs holding capacity). This holds the board securely against vibration while allowing the entire panel to be removed from the bed wall without drilling holes.

### 🔌 CCA & Copper Hybrid Termination Protocols
Because the upper sub-board and vertical umbilical pathways utilize Copper-Clad Aluminum (CCA), specific mechanical precautions are mandatory to counteract aluminum metal creep (cold flow under pressure) and galvanic oxidation in humid environments:

* **Oxide Inhibitor Mandate:** Every bare stripped wire end of a CCA conductor **MUST** be thoroughly coated with a zinc-based anti-oxidant joint compound (e.g., *Noalox* or *Ideal AI-Barrier*) immediately before insertion into any crimp lug, ferrule, or terminal block. This seals out oxygen and humidity to prevent the formation of highly resistive aluminum oxide skins.
* **High-Current Terminals (Victron Orion):** **MUST** utilize bootlace ferrules on all wires entering screw-down tension blocks. Bare CCA strands must never be placed directly under a set screw, as the clamping force will shear the soft aluminum strands. Ferrules must be square- or hex-crimped to maximize surface contact area.
* **Heavy-Gauge Umbilicals & Lugs (8 AWG):** Heavy-duty lugs must be explicitly rated for aluminum/copper transitions (or tin-plated pure copper) and terminated using a **hydraulic hex-crimper** to cold-weld the wire bundle. Hand-squeezing or dimple-crimping CCA will leave internal voids that accelerate corrosion.
* **Short-Circuit Isolation:** All positive lugs are protected with heavy-wall, adhesive-lined polyolefin heat shrink tubing to seal out moisture and provide physical stress relief against mechanical shearing.

### 🔋 Battery Commissioning & Preventive Maintenance Standards
* **Parallel Balancing:** Before connecting the house batteries to the lower sub-board XT60 ports, both batteries **MUST** be charged to 100% State of Charge (SoC) independently. 
* **Visual Labeling:** A permanent visual reminder ("UNITS MUST BE BALANCED TO 100% BEFORE CONNECTION") is mounted adjacent to the XT60 Battery Ports to prevent accidental "Fuse Racing" or BMS trips during hot-swapping.
* **The 6-Month Mechanical Torque Audit:** Due to the physical property of aluminum to slowly creep or deform under continuous mechanical load and trail vibration, all screw terminals on the Orion block and bus bars must be checked and re-torqued to spec exactly 6 months after deployment, and annually thereafter.

---

## 8. Solar PV Wiring & Safety Isolation Protocol

When running the solar panel input wires through the roof gland to the MPPT on the upper board:

### 🔌 1. Physical Specifications
* **Recommended Wire:** Use **10 AWG or 12 AWG PV-rated copper wire** (tray cable). With a max current of $\sim7\text{A}$, this size handles the load with negligible voltage drop and provides excellent weather/UV protection on the roof.
* **PV Overcurrent Protection:** Because a single 200W solar panel is a current-limited source ($I_{sc} \sim 7\text{A}$), it cannot produce enough current to overload 12 AWG or 10 AWG wire. A PV input fuse is not electrically required, but a **10A or 15A inline MC4 fuse** on the roof-side PV (+) line is utilized as an extra physical safety disconnect.

### ⚠️ 2. The "Live-Source" Maintenance Hazard & Roof-Side Protocol
Because a photovoltaic array converts light continuously, the raw positive and negative copper leads running from the panel through the roof gland to the upper sub-board are **live and energized at all times** during daylight hours. 

* **The Operational Vulnerability:** The physical upper-board Solar Disable switch (see `spec/03-controls-settings.md` Section 4) cuts low-current logic power to the MPPT's remote port, cleanly dropping active battery charging to 0A. However, it **does not isolate** the incoming PV conductors. The raw wires landing in the MPPT's `PV +` and `PV -` screw terminals maintain full open-circuit voltage (up to $36.5\text{V}$) whenever the truck is outside.
* **Mandatory Physical Isolation Step:** To prevent arc-flashes, terminal shorts, or damage to the MPPT microprocessor when arranging, servicing, or adjusting port wiring inside the bed, the user **MUST** scale the truck and physically decouple the weatherproof **roof-side MC4 connectors** directly at the panel output leads first. 
* **The Golden Safety Sequence:**
  1. *De-energize Logic:* Toggle the upper sub-board Solar Disable switch to OFF (drops system charging to 0A).
  2. *Sever the Source:* Disconnect the roof-top MC4 plugs outside the SmartCap to make the gland line cold.
  3. *Execute Service:* Perform safe, un-energized terminal handling inside the bed.
