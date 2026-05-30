# Solar Truck Micro-Grid: System Topology Diagram

This diagram visualizes the physical and logical layout of the electrical system, organized by physical location in the 2024 Ford F250 and SmartCap.

```mermaid
graph TD
    %% Location Groups
    subgraph EngineBay ["Engine Bay"]
        StarterBat[Starter Battery]
        SW6Relay[Upfitter SW6 Relay]
        JCase40[40A JCase Fuse]
        IgnitionSrc[Upfitter Ignition Bundle]
    end

    subgraph FrameRails ["Truck Frame Rails (Under Truck)"]
        TJunction[T-Junction Box]
        PosHighway[8 AWG Pos Highway]
        NegHighway[8 AWG Dedicated Neg Highway]
        IgnitionHighway[18 AWG Ignition Line]
    end

    subgraph TailgateArea ["Tailgate / Tail Light Area"]
        TailgateFuse[Tailgate Mini Fuse Panel]
        BedLights[Custom Bed Lights]
        Maxi40A_1[40A Maxi Fuse]
        Maxi40A_2[40A Maxi Fuse]
    end

    subgraph LowerBoard ["Lower Sub-Board (Bed Wall)"]
        Anderson[Anderson Connector]
        MC4[MC4 Logic Connector]
        HousePosBus[House Positive Bus Bar]
        HouseNegBus[Lower Neg Bus Bar]
        XT60_Bat1[XT60 Battery 1 - 20A]
        XT60_Bat2[XT60 Battery 2 - 20A]
        XT60_Load1[XT60 Load 1 - 20A]
        XT60_Load2[XT60 Load 2 - 20A]
        HouseBat1[Goldenmate Battery 1]
        HouseBat2[Goldenmate Battery 2]
    end

    subgraph UpperBoard ["Upper Sub-Board (Ceiling)"]
        TruckPosBus[Truck-Side Pos Bus Bar]
        UpperNegBus[Upper Neg Bus Bar]
        Orion[Victron Orion DC-DC]
        Cyrix[Cyrix-Li-ct Combiner]
        MPPT[Victron SmartSolar MPPT]
        Relay[5-Pin Logic Relay]
        Wago[Wago Logic Splitter]
        SolarSwitch[Solar Disable Switch]
        LogicFuse[2A Logic Fuse]
    end

    subgraph Roof ["Truck Roof"]
        SolarPanel[200W Solar Panel]
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
    
    Relay ---|87a NC| Cyrix
    Relay ---|87 NO| Orion
    
    TruckPosBus --- Orion
    TruckPosBus --- Cyrix
    TruckPosBus --- MPPT
    SolarPanel --- MPPT

    Orion ---|Output +| Cyrix
    Cyrix ---|Terminal 30| HousePosBus
    
    HousePosBus --- XT60_Bat1 --- HouseBat1
    HousePosBus --- XT60_Bat2 --- HouseBat2
    HousePosBus --- XT60_Load1
    HousePosBus --- XT60_Load2
```

## Implementation Notes:
* **Isolated Grounding:** The 8 AWG Dedicated Neg Highway runs directly from the Starter Battery to the Lower Neg Bus Bar, bypassing the vehicle chassis.
* **Fusing Logic:** House battery XT60 ports are fused at 20A (Conservative) with 30A spares available for high-inrush scenarios.
* **Interlocking Relay:** The 5-Pin relay alternates between the Cyrix (Solar/Stationary) and Orion (Alternator/Driving) based on the Upfitter Ignition signal.
