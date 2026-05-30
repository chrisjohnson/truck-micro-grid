# Solar Truck Micro-Grid: System Topology Diagram

This diagram visualizes the physical and logical layout of the electrical system, organized by physical location in the 2024 Ford F250 and SmartCap.

```mermaid
graph TD
    %% Location Groups
    subgraph EngineBay ["Engine Bay"]
        StarterBat["Starter Battery<br/><small>(+ / - Lugs)</small>"]
        SW6Relay["Upfitter SW6 Relay<br/><small>(Output Stud)</small>"]
        JCase40["40A JCase Fuse<br/><small>(In Engine Fuse Box)</small>"]
        IgnitionSrc["Upfitter Ignition Bundle<br/><small>(Engine Bay Harness)</small>"]
    end

    subgraph FrameRails ["Truck Frame Rails (Under Truck)"]
        TJunction["T-Junction Box<br/><small>(8 AWG Splitter)</small>"]
        PosHighway["8 AWG Pos Highway<br/><small>(SW6 Driven)</small>"]
        NegHighway["8 AWG Dedicated Neg Highway<br/><small>(Isolated Return)</small>"]
        IgnitionHighway["18 AWG Ignition Line<br/><small>(Relay Signal)</small>"]
    end

    subgraph TailgateArea ["Tailgate / Tail Light Area"]
        TailgateFuse["Tailgate Mini Fuse Panel<br/><small>(Aux Bus)</small>"]
        BedLights["Custom Bed Lights<br/><small>(16 AWG Strips)</small>"]
        Maxi40A_1["40A Maxi Fuse<br/><small>(Tailgate Side)</small>"]
        Maxi40A_2["40A Maxi Fuse<br/><small>(Sub-Board Side)</small>"]
    end

    subgraph LowerBoard ["Lower Sub-Board (Bed Wall)"]
        Anderson["Anderson Connector<br/><small>(8 AWG SB50)</small>"]
        MC4["MC4 Logic Connector<br/><small>(2-Pin Signal)</small>"]
        HousePosBus["House Positive Bus Bar<br/><small>(Studs 1-5)</small>"]
        HouseNegBus["Lower Neg Bus Bar<br/><small>(Common GND)</small>"]
        XT60_Bat1["XT60 Battery 1<br/><small>(Port 1 - 20A)</small>"]
        XT60_Bat2["XT60 Battery 2<br/><small>(Port 2 - 20A)</small>"]
        XT60_Load1["XT60 Load 1<br/><small>(Fridge - 20A)</small>"]
        XT60_Load2["XT60 Load 2<br/><small>(Aux - 20A)</small>"]
        HouseBat1["Goldenmate Battery 1<br/><small>(100Ah LiFePO4)</small>"]
        HouseBat2["Goldenmate Battery 2<br/><small>(100Ah LiFePO4)</small>"]
    end

    subgraph UpperBoard ["Upper Sub-Board (Ceiling)"]
        TruckPosBus["Truck-Side Pos Bus Bar<br/><small>(Always Hot Bus)</small>"]
        UpperNegBus["Upper Neg Bus Bar<br/><small>(Common GND)</small>"]
        Orion["Victron Orion DC-DC<br/><small>(In+, In-, Out+, Out-, Remote H)</small>"]
        Cyrix["Cyrix-Li-ct Combiner<br/><small>(Term 87, Term 30, Pin 85, Pin 86)</small>"]
        MPPT["Victron SmartSolar MPPT<br/><small>(PV+/-, Bat+/-)</small>"]
        Relay["5-Pin Logic Relay<br/><small>(Pin 30, 86, 85, 87a, 87)</small>"]
        Wago["Wago Logic Splitter<br/><small>(5-Port Lever Nut)</small>"]
        SolarSwitch["Solar Disable Switch<br/><small>(Remote Yellow Wire)</small>"]
        LogicFuse["2A Logic Fuse<br/><small>(Inline Blade)</small>"]
    end

    subgraph Roof ["Truck Roof"]
        SolarPanel["200W Solar Panel<br/><small>(MC4 Connectors)</small>"]
    end

    %% Connections
    StarterBat --- SW6Relay
    SW6Relay --- JCase40
    JCase40 --- PosHighway
    StarterBat --- NegHighway
    IgnitionSrc --- IgnitionHighway

    PosHighway --- TJunction
    TJunction --- Maxi40A_1 --- TailgateFuse --- BedLights
    TJunction --- Maxi40A_2 --- Anderson

    Anderson --- TruckPosBus
    NegHighway --- HouseNegBus --- UpperNegBus
    IgnitionHighway --- MC4 --- Relay

    TruckPosBus --- LogicFuse --- Wago
    Wago --- Relay
    Wago --- SolarSwitch --- MPPT
    
    Relay ---|87a NC -> Pin 85| Cyrix
    Relay ---|87 NO -> Remote H| Orion
    
    TruckPosBus ---|In+| Orion
    TruckPosBus ---|Term 87| Cyrix
    TruckPosBus ---|Bat+| MPPT
    SolarPanel ---|PV +/-| MPPT

    Orion ---|Out+ -> Term 30| Cyrix
    Cyrix ---|Term 30 -> Stud 1| HousePosBus
    
    HousePosBus ---|Stud 2| XT60_Bat1 --- HouseBat1
    HousePosBus ---|Stud 3| XT60_Bat2 --- HouseBat2
    HousePosBus ---|Stud 4| XT60_Load1
    HousePosBus ---|Stud 5| XT60_Load2
```

## Implementation Notes:
* **Isolated Grounding:** The 8 AWG Dedicated Neg Highway runs directly from the Starter Battery to the Lower Neg Bus Bar, bypassing the vehicle chassis.
* **Fusing Logic:** House battery XT60 ports are fused at 20A (Conservative) with 30A spares available for high-inrush scenarios.
* **Interlocking Relay:** The 5-Pin relay alternates between the Cyrix (Solar/Stationary) and Orion (Alternator/Driving) based on the Upfitter Ignition signal.
