# Mobile Electrical & Solar Micro-Grid: Master Design & Verification Spec
**System Owner:** Christopher Johnson  
**Platform:** 2024 Ford F250 Lariat Tremor (7.3L Gas, SmartCap Bed Camper Shell)  
**Target File Path:** `file:///Users/cjohnson/src/gemini/project-solar-truck/master_design_spec.md`

---

## 1. System Overview & Parasitic Drain Resolution

This system is a dual-battery, solar-assisted 12V DC micro-grid spanning the engine bay, the truck frame, and the truck bed.

### ⚠️ Parasitic Drain Analysis
In the initial setup, the main 8 AWG power highway to the tailgate was switched by **Upfitter Switch 6 (SW6)**, which was modified to be "hot-at-all-times". 
* **The Issue:** Leaving SW6 toggled ON to keep the SmartCap lights active when camping kept the factory upfitter relay coil energized continuously. The relay coil draws **$\sim150\text{–}200\text{mA}$** of holding current. This parasitic draw, combined with the truck's factory connectivity modules, drained the starting batteries during short periods of inactivity.
* **The Resolution:** 
  1. The main **8 AWG CCA highway** is moved off the Upfitter relay and connected **directly to the starter battery positive terminal** (fused at **30A** in the engine bay).
  2. The tailgate bus bar is now **always hot**, allowing the custom SmartCap LED light strips (referred to as **truck bed lights**) to run directly off the vehicle starting battery network without upfitter relay coil drain. These custom lights are controlled by a local toggle switch with an always-on LED, which is distinct from the F250's OEM factory bed lights that feature an automatic timeout.
  3. The SW6 fuse in the engine compartment is restored to its factory **ignition-switched** position. Its low-current output is used purely as an **ignition signal wire (18 AWG)** to trigger the control logic at the tailgate.

---

## 2. Core Hardware Inventory & Specifications

### Power Sources & Buffers:
* **House Bank (Removable):** 2x Goldenmate 100Ah Lithium Iron Phosphate (LiFePO4) batteries (sit on the truck bed floor).
  * *Internal Protection:* Integrated BMS with Low-Voltage Disconnect (LVD) threshold $\le 11.0\text{V}$.
  * *Local Fusing:* Each battery box connects via a **12 AWG 24-inch XT60-to-XT60 cable** to the lower sub-board.
  * *Panel Fusing:* Each house battery XT60 port is individually fused at **30A** on the house positive bus bar.
* **Starter Bank:** 2024 Ford F250 OEM dual starting battery network.
* **Solar Array:** 1x Renogy 200W Shadowflux High-Efficiency N-Type Panel.
  * *Max Voltage ($V_{mp}$):* $31.3\text{V}$
  * *Open-Circuit Max Voltage ($V_{oc}$):* $36.5\text{V}$
  * *Max Current ($I_{sc}$):* $\sim7\text{A}$

### Management & Control Electronics:
* **DC-DC Charger:** Victron Orion-Tr Smart 12/12-18A (18A continuous output).
* **Battery Combiner:** Cyrix-Li-ct Intelligent Voltage-Sensing Combiner (bridges at $\ge 13.4\text{V}$, opens at $< 12.8\text{V}$).
* **Solar Charge Controller:** Victron SmartSolar MPPT (Mounted on the upper sub-board, wired to the **truck-side** positive bus bar).
* **Logic Controller:** 5-Pin SPDT Automotive Relay (12V, 30/40A rated).

---

## 3. Physical Layout & Wiring Topology

To support easy removal of the SmartCap, the system is divided into three distinct zones connected by quick-disconnects.

```
                                      [ ENGINE BAY ] 
                                             │
                                     (30A MIDI Fuse)
                                             │
                             [ 8 AWG CCA Frame Run (Under Truck) ]
                                             │
                                    (T-Junction Box)
                      ┌──────────────────────┴──────────────────────┐
                      ▼                                             ▼
        [ Tailgate Mini Fuse Panel ]                      [ Anderson Connector (8 AWG OFC) ]
          (SmartCap LED Light Strips)                     [ MC4 Connector (18 AWG Ignition) ]
                                                                    │
                                                           [ LOWER SUB-BOARD ]
                                                          (Wall above tub lip)
                                                                    │
                                                 ┌──────────────────┴──────────────────┐
                                                 ▼                                     ▼
                                       (8 AWG OFC + 18 AWG Run)              (House Positive Bus Bar)
                                       (Runs up the SmartCap wall)             │─ 30A Fuse -> XT60 Port 1 (Battery 1)
                                                 │                             │─ 30A Fuse -> XT60 Port 2 (Battery 2)
                                                 ▼                             │─ 20A Fuse -> XT60 Load 1 (Fridge/Heat)
                                        [ UPPER SUB-BOARD ]                    │─ 20A Fuse -> XT60 Load 2 (Chargers)
                                      (SmartCap Ceiling/Wall)                  └─ House Battery Negatives
                                                 │
                             ┌───────────────────┼───────────────────┐
                             ▼                   ▼                   ▼
                     [ Truck Bus Bar ]   [ 5-Pin Relay ]    [ Upper Negative Bus Bar ]
```

### 🎛️ Sub-Board Details:

#### 1. Lower Sub-Board (Wall above tub lip):
* **Grounding:** The lower common negative bus bar connects directly to the incoming 8 AWG starting battery ground run, the upper common negative bus bar, and the house battery negatives.
* **Distribution:** Houses the XT60 ports for the house batteries (each fused at **30A**) and load ports (each fused at **20A**).
* **Charging Path:** The final positive terminal on the house-side bus bar (Stud 1) connects to the **8 AWG OFC House Charging Highway** wire running up to the upper sub-board.

#### 2. Upper Sub-Board (SmartCap Ceiling):
* **Truck-Side Positive Bus Bar:** Connects to the incoming always-hot 8 AWG OFC positive line from the Anderson connector.
* **Solar Controller (MPPT) Connections:**
  * Battery (+) runs via a **12 AWG wire** to the truck-side positive bus bar (protected by a **20A MIDI fuse**).
  * Battery (-) runs via a **12 AWG wire** to the upper common negative bus bar.
  * *Logic:* Solar power flows through the truck bus bar and the Anderson/T-junction back to charge the F250's starter batteries first.
* **Combiner (Cyrix) Starter-Side Connection:**
  * Terminal 87 connects to the truck-side positive bus bar via an **8 AWG CCA wire** (protected by a **30A MIDI fuse**).
* **DC-DC Charger (Orion) Input Connection:**
  * Input (+) connects to the truck-side positive bus bar via an **8 AWG CCA wire** (protected by a **40A MIDI fuse**).
  * Input (-) and Output (-) are wired to the upper negative bus bar using **8 AWG CCA**.
* **House Highway Conjunction (Cyrix Terminal 30 / Orion Output+):**
  * The **8 AWG OFC House Charging Highway** wire coming up from the lower board lands directly on **Cyrix Terminal 30**.
  * The Orion Output (+) (8 AWG) lands on this same **Cyrix Terminal 30** lug via a local copper jumper.
  * *Logic:* Power flowing from either the Orion (engine running) or the Cyrix (solar active & starter battery $> 13.4\text{V}$) feeds directly into Cyrix Terminal 30 and travels down the 8 AWG OFC wire to the house positive bus bar.

---

## 4. Control Logic & Switch Configurations

### 🔌 Low-Current Logic Power (2A Blade Fuse & Wago)
To supply logic and remote control power safely:
* A **2A inline blade fuse** taps off the truck-side positive bus bar and feeds a **Wago connector**.
* The Wago connector splits this 12V feed to:
  1. **Pin 30 (SPDT Relay Input):** Provides control voltage to the interlocking relay loop.
  2. **Solar On/Off Switch:** An inline switch that controls the VE.Direct yellow remote enable wire plugged into the MPPT. Flipping this switch OFF immediately drops MPPT generation to 0A, allowing you to isolate the bed grid for battery box maintenance or storage.

### 🎛️ Relay Pinout & Wiring Schedule (SPDT Relay)
The logic wires utilize a mix of **18 AWG and 12 AWG OFC primary wire** matching the component terminals.

* **Pin 30 (Common Input Power):** Red wire from the Wago connector (fused at 2A).
* **Pin 86 (Relay Coil Positive +):** 18 AWG Ignition signal wire coming up from the lower board's MC4 connector (powered by SW6 when ignition is ON).
* **Pin 85 (Relay Coil Ground -):** Black wire leading directly to the **Upper Common Negative Bus Bar**.
* **Pin 87a (Normally Closed - NC Output):** Green/Orange wire leading to the **Combiner Toggle Switch** (described below).
* **Pin 87 (Normally Open - NO Output):** Purple/White wire leading to the green **Remote H-Pin Terminal Block** on the **Victron Orion DC-DC Charger**.

### 🎛️ Combiner Toggle Switch (Camp Mode Switch)
To prevent the Cyrix combiner solenoid from drawing continuous holding current ($\sim220\text{–}300\text{mA}$) at night when not camping, or when the vehicle is parked long-term:
* A low-current toggle switch is placed inline with the **Green/Orange wire** running from SPDT Relay Pin 87a to **Cyrix Terminal 85**.
* **Switch OFF (Daily/Storage Mode):** Disconnects the Cyrix control logic. The Cyrix is forced open and completely disabled. Solar charging is restricted *only* to the starting batteries. There is zero risk of starter battery depletion from combiner coil draw or backfeeding house loads.
* **Switch ON (Camp Mode):** Arms the Cyrix control logic. The system behaves automatically, bridging to charge the house batteries once starting batteries are topped off ($>13.4\text{V}$), keeping camp loads (fridge) running.

### 🔄 Operational Truth Table
| Ignition State (SW6) | Combiner Switch | Relay Pin 30 Connects To | Cyrix Combiner State | Orion DC-DC State | System Behavior |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Key OFF / SW6 OFF** | **OFF** | **Pin 87a (NC)** | **Forced OPEN** | **Forced OFF** | Daily/Storage. Solar maintains starter batteries. House bank is completely isolated. Zero standby draw from Cyrix coil. |
| **Key OFF / SW6 OFF** | **ON** | **Pin 87a (NC)** | **Armed / Active** | **Forced OFF** | Camp Mode. Solar charges starter; once starter hits $>13.4\text{V}$, Cyrix bridges to charge house bank and run fridge. Disconnects automatically if combined voltage drops below $12.8\text{V}$ at night. |
| **Key ON / SW6 ON** | **Either** | **Pin 87 (NO)** | **Forced OPEN** | **Armed / Active** | Engine running. Alternator charges starter. Orion charges house bank via 18A highway. Cyrix is locked open to prevent bypass loops. |

---

## 5. Main House Panel Power Distribution & Fusing

The lower sub-board house positive bus bar distributes charging current and handles load fusing:

```
                          [ HOUSE POSITIVE BUS BAR ]
      _______________________/_______|_____________________________________
     |               |               |               |               |
 [ STUD 1 ]      [ STUD 2 ]      [ STUD 3 ]      [ STUD 4 ]      [ STUD 5 ]
   (Input)         (30A)           (30A)           (20A)           (20A)
     |               |               |               |               |
  Incoming       XT60 House      XT60 House       XT60 Load       XT60 Load
Charging Hwy     Battery 1       Battery 2        Port 1          Port 2
```

### 🎛️ Board Configuration
* **Stud 1: House Charging Highway (No Local Fuse - Fed by Upper Board)**
  * *Wiring:* 8 AWG OFC positive wire running down from Cyrix Terminal 30 on the upper board.
* **Stud 2: XT60 House Battery 1 (30A Fuse)**
  * *Wiring:* 12 AWG pure copper pigtail to XT60 port. Connected to battery box via a 24" 12 AWG XT60-to-XT60 patch cable.
* **Stud 3: XT60 House Battery 2 (30A Fuse)**
  * *Wiring:* 12 AWG pure copper pigtail to XT60 port. Connected to battery box via a 24" 12 AWG XT60-to-XT60 patch cable.
  * *Safety Guardrail:* Cross-battery isolation is enforced because inter-battery balancing current must pass through *both* Stud 2 and Stud 3 fuses to cross-feed.
* **Stud 4: XT60 Load Port 1 (20A Fuse)**
  * *Wiring:* 12 AWG pure copper. (Camper fridge or diesel heater).
* **Stud 5: XT60 Load Port 2 (20A Fuse)**
  * *Wiring:* 12 AWG pure copper. (USB-C hubs, lighting, etc.).

---

## 6. System Physics & Impedance Verification

### The 50-Foot Circuit Loop Impedance
The main charging run consists of a **50-foot round-trip run of 8 AWG CCA wire** along the frame, transitioning to **8 AWG OFC wire** into the bed, and terminating at the sub-boards.
* **Total Loop Resistance:** **$\sim0.08\ \Omega$** (lowered by the use of 8 AWG OFC for the bed/SmartCap runs, pigtails, and low-resistance Anderson connectors).
* **Current-Limiting Safety Profile:** Due to this line resistance, Ohm's Law limits the maximum current that can transfer between the starter and house batteries during unmanaged balancing loops:
  $$\Delta V = 30\text{A} \times 0.08\ \Omega = 2.4\text{V}$$
* **Nuisance Tripping Prevention:** To draw a fuse-blowing 30A from the starting bank during a Cyrix bridge event, the house bank voltage would have to sit below $11.0\text{V}$ ($13.4\text{V} - 2.4\text{V} = 11.0\text{V}$). Because the Goldenmate internal BMS activates its low-voltage cut-off at $\le 11.0\text{V}$, nuisance fuse-blowing from battery-to-battery balancing inrushes is physically impossible.

---

## 7. Mechanical Installation & Mounting Standards

### 🔩 Board Mounting
* **Upper Sub-Board (SmartCap Ceiling):** Bolted securely to the SmartCap's integrated ceiling **M8 threaded inserts** using lock washers and fender washers to resist vibration during off-roading.
* **Lower Sub-Board (Tub Lip Wall):** Mounted using **4x 66lb rubberized neo-magnets** (providing a total of 264 lbs holding capacity). This holds the board securely against vibration while allowing the entire panel to be removed from the bed wall without drilling holes.

### 🔌 Wire Management & Route Dressing
* **Wall Runs:** Wires running between the lower and upper sub-boards are dressed along the SmartCap wall using **strong routing magnets** to keep them secure and flat against the shell.
* **External Protection:** All wires running along the F250 frame-rail and within the engine bay are **100% protected in corrugated wire loom** to guard against abrasion, heat, and road debris.
* **Termination Standards:**
  * Heavy-gauge wires (8 AWG) utilize pure copper heavy-duty lugs terminated with a **hydraulic hex-crimper**.
  * Small-gauge wires (18 AWG / 12 AWG) utilize premium mini ring terminals.
  * All positive lugs are protected with **adhesive-lined heat shrink tubing** for complete short-circuit isolation.

---

## 8. Future Solar PV Wiring Guidance

When running the solar panel input wires through the roof gland to the MPPT on the upper board:
* **Recommended Wire:** Use **10 AWG or 12 AWG PV-rated copper wire** (tray cable). With a max current of $\sim7\text{A}$, this size handles the load with negligible voltage drop and provides excellent weather/UV protection on the roof.
* **PV Overcurrent Protection:** Because a single 200W solar panel is a current-limited source ($I_{sc} \sim 7\text{A}$), it cannot produce enough current to overload 12 AWG or 10 AWG wire. A PV input fuse is not electrically required, but a **10A or 15A inline MC4 fuse** on the roof-side PV (+) line is recommended as an extra physical safety disconnect.
