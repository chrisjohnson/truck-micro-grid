# Solar Truck Micro-Grid: System Topology Diagram

This diagram visualizes the physical and logical layout of the electrical system, organized by physical location in the 2024 Ford F250 and SmartCap.

```mermaid
graph TD
    %% Location Groups
    subgraph EngineBay ["Engine Bay"]
        StarterBat["Starter Battery<br/><small>(+ / - Lugs)</small>"]
        SW6Relay["Upfitter SW6 Relay<br/><small>(Output Stud)</small>"]
        JCase40["40A JCase Fuse"]
        IgnitionSrc["Upfitter Ignition Bundle"]
    end

    subgraph FrameRails ["Truck Frame Rails (Under Truck)"]
        PosTJunction["Positive T-Junction Box"]
        NegTJunction["Negative T-Junction Box"]
        PosHighway["8 AWG Pos Highway"]
        NegHighway["8 AWG Dedicated Neg Highway"]
        IgnitionHighway["18 AWG Ignition Line"]
        Maxi40A_1["40A Maxi Fuse (Tail Light Branch)"]
        Maxi40A_2["40A Maxi Fuse (Bed Branch)"]
    end

    subgraph TailLightArea ["Tail Light Cavity"]
        TailLightFuse["Tail Light Mini Fuse Panel"]
        TailLightNeg["Tail Light Neg Bus"]
        BedLights["Custom Bed Lights"]
    end

    subgraph LowerBoard ["Lower Sub-Board (Bed Wall)"]
        Anderson["Anderson Connector"]
        MC4["MC4 Logic Connector"]
        HousePosBus["House Positive Bus Bar"]
        HouseNegBus["Lower Neg Bus Bar"]
        XT60_Bat1["XT60 Battery 1"]
        XT60_Bat2["XT60 Battery 2"]
        XT60_Load1["XT60 Load 1"]
        XT60_Load2["XT60 Load 2"]
        HouseBat1["Goldenmate Battery 1"]
        HouseBat2["Goldenmate Battery 2"]
    end

    subgraph UpperBoard ["Upper Sub-Board (Ceiling)"]
        TruckPosBus["Truck-Side Pos Bus Bar"]
        UpperNegBus["Upper Neg Bus Bar"]
        Orion["Victron Orion DC-DC"]
        Cyrix["Cyrix-Li-ct Combiner"]
        MPPT["Victron SmartSolar MPPT"]
        Relay["5-Pin Logic Relay"]
        Wago["Wago Logic Splitter"]
        SolarSwitch["Solar Disable Switch"]
        LogicFuse["2A Logic Fuse"]
    end

    subgraph Roof ["Truck Roof"]
        SolarPanel["200W Solar Panel"]
    end

    %% Connections with precise terminal labels
    StarterBat --- SW6Relay
    SW6Relay --- JCase40
    JCase40 --- PosHighway
    StarterBat --- NegHighway
    IgnitionSrc --- IgnitionHighway

    PosHighway --- PosTJunction
    PosTJunction --- Maxi40A_1 --- TailLightFuse --- BedLights
    PosTJunction --- Maxi40A_2 --- Anderson

    Anderson --- TruckPosBus
    NegHighway --- NegTJunction
    NegTJunction --- TailLightNeg
    NegTJunction ---|Neg Highway| HouseNegBus
    HouseNegBus --- UpperNegBus
    IgnitionHighway --- MC4
    
    MC4 ---|Relay Pin 86| Relay
    LogicFuse ---|Wago Input| Wago
    TruckPosBus --- LogicFuse
    
    Wago ---|Relay Pin 30| Relay
    Wago ---|Switch Input| SolarSwitch
    SolarSwitch ---|Remote Yellow| MPPT
    
    Relay ---|Pin 87a NC > Cyrix Pin 85| Cyrix
    Relay ---|Pin 87 NO > Orion Remote H| Orion
    
    TruckPosBus ---|Orion In+| Orion
    TruckPosBus ---|Cyrix Term 87| Cyrix
    TruckPosBus ---|MPPT Bat+| MPPT
    SolarPanel ---|MPPT PV+/-| MPPT

    Orion ---|Orion Out+ > Cyrix Term 30| Cyrix
    Cyrix ---|Cyrix Term 30 > Stud 1| HousePosBus
    
    HousePosBus ---|Stud 2| XT60_Bat1 --- HouseBat1
    HousePosBus ---|Stud 3| XT60_Bat2 --- HouseBat2
    HousePosBus ---|Stud 4| XT60_Load1
    HousePosBus ---|Stud 5| XT60_Load2
```

## Implementation Notes:
* **Isolated Grounding:** The 8 AWG Dedicated Neg Highway runs directly from the Starter Battery to the Lower Neg Bus Bar, bypassing the vehicle chassis.
* **Fusing Logic:** House battery XT60 ports are fused at 20A (Conservative) with 30A spares available for high-inrush scenarios.
* **Interlocking Relay:** The 5-Pin relay alternates between the Cyrix (Solar/Stationary) and Orion (Alternator/Driving) based on the Upfitter Ignition signal.
