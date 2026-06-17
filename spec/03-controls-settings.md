# Control Logic & VictronConnect Settings

**Cross-reference:** This file combines Sections 4 and 13 of the master spec. For the physical relay pinout in context of the topology, see `spec/01-system-core.md` (Section 3). For wiring of control wires, see `spec/02-wiring-schedule.md`.

---

## 4. Control Logic & Switch Configurations

### 🔌 Low-Current Logic Power (2A Blade Fuse & Wago)
To supply logic and remote control power safely:
* A **2A inline blade fuse** taps off the truck-side positive bus bar and feeds a **Wago connector**.
* The Wago connector splits this 12V feed to:
  1. **Pin 30 (SPDT Relay Input):** Provides control voltage to the interlocking relay loop.
  2. **Solar On/Off Switch:** An inline switch that controls a 12V feed to a **Victron VE.Direct Non-Inverting Remote On/Off Cable**. When the switch is toggled ON, 12V is supplied to the yellow cable terminal, signaling the MPPT's digital microprocessor to enable solar generation. Flipping this switch OFF drops the signaling voltage to 0V, forcing the MPPT to a "Remote Disabled" standby state (0A output) for grid maintenance or storage.

### 🎛️ Relay Pinout & Wiring Schedule (SPDT Relay)
The logic wires utilize a mix of **18 AWG and 12 AWG OFC primary wire** matching the component terminals.

* **Pin 30 (Common Input Power):** Red wire from the Wago connector (fused at 2A).
* **Pin 86 (Relay Coil Positive +):** 18 AWG Orange wire coming up from the lower board's MC4 connector, tapped from the **factory Ford upfitter ignition signal bundle** in the engine bay (hot only when key is in ON/RUN position, independent of SW6 state).
* **Pin 85 (Relay Coil Ground -):** Black wire leading directly to the **Upper Common Negative Bus Bar** (Pin 85).
* **Pin 87a (Normally Closed - NC Output):** Green/Orange wire leading directly to **Terminal 85 (Control)** on the **Cyrix-Li-ct Combiner**.
* **Pin 87 (Normally Open - NO Output):** Purple/White wire leading to the green **Remote H-Pin Terminal Block** on the **Victron Orion DC-DC Charger**.

### 🔄 Operational Truth Table

> [!NOTE]
> SW6 (always hot) and the ignition key signal are **independent inputs**. SW6 controls the main 8 AWG power highway to the T-junction and is always energized. The relay coil at Pin 86 is triggered solely by the factory upfitter ignition bundle — it is hot only when the key is in the ON/RUN position, regardless of SW6 state.

| Ignition Key State | Relay Coil (Pin 86) | Relay Pin 30 Connects To | Cyrix Combiner State | Orion DC-DC State | System Behavior |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Key OFF** | De-energized | **Pin 87a (NC)** | **Armed / Active** | **Forced OFF** | SW6 highway always hot. Solar charges starter battery if the VE.Direct remote switch is enabled. When starter hits $\ge 13.4\text{V}$, Cyrix bridges to charge/maintain house bank and run fridge. Auto-disconnects if combined voltage drops below $12.8\text{V}$ (e.g. at night under load). |
| **Key ON/RUN** | Energized | **Pin 87 (NO)** | **Forced OPEN** | **Armed / Active** | Alternator charges starter via SW6 highway. Orion charges house bank at 18A. Cyrix is locked open to prevent alternator overload or uncontrolled bypass loops. |

---

## 13. Non-Default App Programming & Software Configurations

This section formalizes the custom programming targets required within the VictronConnect app interface. These overrides actively compensate for the accepted project infrastructure debts (see `spec/01-system-core.md` Section 12), ensuring optimal coordination across varying seasonal profiles.

### 🔌 1. Victron Orion-Tr Smart 12/12-18A Settings
By default, the Orion ships in "Power Supply" mode. It must be toggled to **"Charger"** mode immediately. Because it operates over the 50-foot 8 AWG CCA highway run paired with 8 AWG CCA vertical umbilical runs, its voltage-sensing parameters must be de-rated to prevent premature engine-shutdown cycling.

#### ⚙️ Engine Shutdown Detection (Advanced Settings)
* **Alternator Type:** Smart Alternator
* **Start Voltage:** `14.0V`
  * *Justification:* The Ford 7.3L Godzilla smart alternator rests the starting battery at $\sim13.5\text{V}$ but spikes to $\sim14.5\text{V}$ upon crank. A 14.0V target ensures the Orion only pulls power when the alternator is spinning.
* **Delayed Start Voltage:** `13.5V`
* **Delayed Start Voltage Delay:** `60 seconds`
* **Shutdown Voltage:** `13.1V`
  * *Justification:* Raised slightly from the factory default ($13.0\text{V}$) to account for the voltage drop across the 8 AWG CCA highway run under load. This guarantees the Orion shuts down cleanly when the truck engine turns off, preventing starting bank draw down.

#### ⚙️ Input Voltage Lock-Out Settings
* **Input Voltage Lock-Out:** Enabled
* **Lock-out Threshold (Under-Voltage):** `12.7V` (or `12.6V` if line drop triggers unexpected dropouts)
  * *Justification:* Crucial non-default override. Compensates for the increased upper-board ground shift caused by the compound CCA returns. If the ground shifts up, a lower lock-out is mandatory to ensure the Orion doesn't cut out prematurely under full continuous alternator pull.
* **Restart Threshold:** `13.2V`

#### 🔋 Charge Profile Settings (User-Defined Custom)
* **Absorption Voltage:** `14.4V` (Adaptive, Max Duration: `2 hours`)
* **Float Voltage:** `13.5V`
* **Re-bulk Voltage Offset:** `0.10V` (Initiates re-bulk if house bank drops below $13.4\text{V}$)

---

### ☀️ 2. Victron SmartSolar MPPT Settings
Because your 200W panel is flat-mounted and wired directly to the *truck-side* positive bus bar, its primary directive is floating the starting battery at home while maintaining the armed logic circuits. 

#### 🔋 Battery & Charge Optimization Parameters
* **Battery Preset:** User-Defined (Do not use "Smart Lithium" default due to target offsets)
* **Max Charge Current:** `20A` (Full controller capacity)
* **Absorption Voltage:** `14.55V` 
  * *Justification:* Shifted upward by **$+0.15\text{V}$** over standard lithium parameters to offset the shared 12 AWG CCA local battery line resistance and vertical ground wire drop during active windows, ensuring that the true voltage hitting the battery boxes resolves to a perfect $14.4\text{V}$.
* **Float Voltage:** `13.65V`
  * *Justification:* Boosted slightly above standard profiles. Since the solar array hits the lead-acid starting battery first, this higher float target accounts for local line drops, keeps the lead-acid bank properly saturated, and counteracts the baseline $175\text{mA}$ SW6 upfitter relay coil debt.
* **Equalization Charge:** Disabled (Ensure toggle is physically grayed out; equalization will destroy LiFePO4 chemistry)

#### ⚙️ VE.Direct Port / RX Pin Digital Logic Override
* **TX/RX Port Configuration Menu:** Enabled
* **RX Pin Function Assignment:** `Remote On/Off`
  * *Justification:** **CRITICAL MANDATORY OVERRIDE.** By default, the MPPT treats the physical VE.Direct RX line as a pure digital data pin. This parameter must be manually reassigned to instruct the internal microprocessor to read the yellow cable's voltage state as a physical high/low ignition signal. Once active, a 12V positive signal at the RX pin authorizes charging, while a 0V signal hard-drops controller generation to 0A, forcing a "Remote Disabled" standby code.

#### ❄️ Low-Temperature Cut-off Behavior
* **Low-Temperature Cut-off:** `Disabled`
  * *Justification:* Because the MPPT is wired to the *truck-side* positive bus bar, its local temperature sensor measures the canopy/ceiling climate. If left enabled, freezing winter temperatures in Ohio would cause the MPPT to stop generating entirely. This would cut off the solar-maintenance loop to your starter battery. 
  * *Safety Override:* The Goldenmate battery boxes contain their own internal hardware BMS units with independent Low-Temperature Disconnects ($\le 32^\circ\text{F}\ / \ 0^\circ\text{C}$). If the canopy freezes, the solar panel can safely continue maintaining the lead-acid starter battery, while the lower battery boxes protect themselves via their built-in BMS switches.
