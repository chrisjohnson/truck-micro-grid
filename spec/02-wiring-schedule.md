# Wiring Schedule — Comprehensive Point-to-Point Connection Map

**Cross-reference:** This is Section 9 of the master spec. Fuse audit details in `spec/01-system-core.md` (Section 10). Fuse values in this table are the installed values; see the fuse audit for coordination rationale.

| Wire/Circuit Name | From (Component / Term) | To (Component / Term) | Gauge & Type | Fuse (Type / Rating) | Physical Location & Routing |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Main Engine Pos Highway** | Upfitter SW6 Relay Output Stud | Tailgate Frame T-Junction Box | 8 AWG CCA | SW6 Fuse / 40A | Relay Box -> Frame Rail under truck |
| **Main Engine Neg Highway** | Starter Battery (-) Lug | Frame Negative T-Junction Box | 8 AWG CCA | Unfused | Dedicated wire run: Engine Bay -> Frame Rail |
| **Tail Light Neg Branch** | Frame Negative T-Junction Box | Tail Light Negative Bus | 8 AWG CCA | Unfused | Frame Rail -> Tail Light Cavity |
| **Bed Negative Branch** | Frame Negative T-Junction Box | Anderson Connector (Input -) | 8 AWG CCA | Unfused | Frame Rail -> Bed Floor |
| **Tail Light Pos Branch** | Frame Positive T-Junction Box | Tail Light Mini Fuse Panel | 8 AWG CCA | Maxi / 40A (at junction) | Frame Rail -> Tail Light Cavity |
| **Custom Truck Bed Lights** | Tail Light Mini Fuse Panel (+) / Neg Bus (-) | LED Strips / Toggle Switch | 16 AWG OFC | ATO / 5A or 10A | Tail Light Cavity interior |
| **Bed Feed Positive** | Frame Positive T-Junction Box | Anderson Connector (Input +) | 8 AWG OFC | Maxi / 40A (at junction) | Frame Rail -> bed floor entry point |
| **Ignition Signal Line** | Factory Upfitter Ignition Bundle (Engine Bay) | MC4 Connector (Input +) | 18 AWG OFC (Orange) | OEM Upfitter Fuse (engine bay fuse box) | Engine Bay bundle -> Frame Rail -> Bed Floor |
| **Truck-Side Highway Pos** | Anderson Connector (Output +) | Truck-Side Pos Bus Bar (Upper) | 8 AWG CCA | Unfused | Bed Floor -> Upper Sub-Board |
| **System Negative Highway** | Anderson Connector (Output -) | Lower Neg Bus Bar (Lower) | 8 AWG CCA | Unfused | Bed Floor -> Lower Sub-Board |
| **Inter-Board Negative** | Lower Neg Bus Bar (Lower) | Upper Neg Bus Bar (Upper) | 8 AWG CCA | Unfused | Lower Board -> up SmartCap wall |
| **Orion Power Input** | Truck-Side Pos Bus Bar (Upper) | Orion-Tr Smart Input (+) | 8 AWG CCA | MIDI / 40A | Upper Sub-Board |
| **Cyrix Starter Line** | Truck-Side Pos Bus Bar (Upper) | Cyrix Combiner Terminal 87 | 8 AWG OFC | MIDI / 30A | Upper Sub-Board |
| **MPPT Power Output** | Victron SmartSolar MPPT Bat (+) | Truck-Side Pos Bus Bar (Upper) | 12 AWG CCA | MIDI / 20A | Upper Sub-Board |
| **Logic Power Feed** | Truck-Side Pos Bus Bar (Upper) | Wago Connector Input | 18 AWG OFC | Inline Blade / 2A | Upper Sub-Board |
| **MPPT Solar Disable Switch** | Wago Connector Output | MPPT Remote H-pin (Yellow wire) | 18 AWG OFC | Unfused (In 2A Loop) | Upper Sub-Board |
| **Relay Pin 30 logic power** | Wago Connector Output | Relay Pin 30 | 18 AWG OFC | Unfused (In 2A Loop) | Upper Sub-Board |
| **Relay Coil Control** | MC4 Connector Output | Relay Pin 86 | 18 AWG OFC (Orange) | Unfused | MC4 (Lower) -> parallel up wall |
| **Relay Coil Ground** | Relay Pin 85 | Upper Neg Bus Bar (Upper) | 18 AWG OFC (Black) | Unfused | Upper Sub-Board |
| **Cyrix Control Terminal** | Relay Pin 87a (NC) | Cyrix Combiner Terminal 85 | 18 AWG OFC (Grn/Org) | Unfused (In 2A Loop) | Upper Sub-Board |
| **Orion Control Terminal** | Relay Pin 87 (NO) | Orion Remote H-Pin | 18 AWG OFC (Pur/Wht) | Unfused (In 2A Loop) | Upper Sub-Board |
| **Orion Ground** | Orion Input (-) and Output (-) | Upper Neg Bus Bar (Upper) | 8 AWG CCA | Unfused | Upper Sub-Board |
| **Cyrix Ground** | Cyrix Combiner Terminal 86 | Upper Neg Bus Bar (Upper) | 18 AWG OFC (Black) | Unfused | Upper Sub-Board |
| **MPPT Ground** | Victron SmartSolar MPPT Bat (-) | Upper Neg Bus Bar (Upper) | 12 AWG CCA | Unfused | Upper Sub-Board |
| **Orion Output Jumper** | Orion-Tr Smart Output (+) | Cyrix Combiner Terminal 30 | 8 AWG CCA | Unfused | Upper Sub-Board |
| **House Charging Highway** | Cyrix Combiner Terminal 30 | House Positive Bus Bar Stud 1 (Lower) | 8 AWG OFC | MIDI / 30A | Upper Board -> down SmartCap wall |
| **House Battery 1 Charge Power** | House Positive Bus Bar Stud 2 | XT60 Battery 1 Charge Port | 12 AWG OFC | MIDI / 30A | Lower Sub-Board |
| **House Battery 1 Load Power** | House Positive Bus Bar Stud 3 | XT60 Battery 1 Load Port | 12 AWG OFC | MIDI / 30A | Lower Sub-Board |
| **House Battery 2 Charge Power** | House Positive Bus Bar Stud 4 | XT60 Battery 2 Charge Port | 12 AWG OFC | MIDI / 30A | Lower Sub-Board |
| **House Battery 2 Load Power** | House Positive Bus Bar Stud 5 | XT60 Battery 2 Load Port | 12 AWG OFC | MIDI / 30A | Lower Sub-Board |
| **House Load Port 1 Power** | House Positive Bus Bar Stud 6 | XT60 Load Port 1 | 12 AWG OFC | MIDI / 20A | Lower Sub-Board |
| **House Load Port 2 Power** | House Positive Bus Bar Stud 7 | XT60 Load Port 2 | 12 AWG OFC | MIDI / 20A | Lower Sub-Board |
| **House Battery 1 Charge Patch** | Lower Board XT60 Battery 1 Charge Port | Battery Box 1 Charge/House Port | 12 AWG OFC | Unfused (Protected by panel MIDI + Goldenmate BMS) | Bed floor (30-inch XT60 patch) |
| **House Battery 2 Charge Patch** | Lower Board XT60 Battery 2 Charge Port | Battery Box 2 Charge/House Port | 12 AWG OFC | Unfused (Protected by panel MIDI + Goldenmate BMS) | Bed floor (30-inch XT60 patch) |
| **House Battery 1 Load Patch** | Lower Board XT60 Battery 1 Load Port | Battery Box 1 Loads Only Port | 12 AWG OFC | Unfused (Protected by panel MIDI + BatteryProtect + BMS) | Bed floor (30-inch XT60 patch) |
| **House Battery 2 Load Patch** | Lower Board XT60 Battery 2 Load Port | Battery Box 2 Loads Only Port | 12 AWG OFC | Unfused (Protected by panel MIDI + BatteryProtect + BMS) | Bed floor (30-inch XT60 patch) |
| **Solar Panel Input (Future)** | Solar Panel on Roof | Victron SmartSolar MPPT PV (+/-) | 10 or 12 AWG PV | Optional MC4 / 10A-15A | Roof -> Gland -> Upper Board |
