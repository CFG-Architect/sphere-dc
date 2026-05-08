# heat-export.md

# SPHERE-DC Technical Document 07: Heat Export

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
Heat Export = external transfer of usable thermal energy from SPHERE-DC to a defined heat consumer

Concept baseline:

SPHERE-DC may export remaining usable heat after higher-priority internal modules are served.
Exported heat counts as useful only if it replaces real external heat demand.
Heat export must not be counted as heat elimination.
If no external demand exists, available heat must route to thermal buffers or thermal tail.
```

1. Scope

This document defines calculation logic for:

available export heat
external heat demand
temperature suitability
heat transfer interface
pipeline / distribution losses
delivered heat
useful exported heat
pump power
daily exported energy
demand matching
fallback routing
failure and bypass conditions

This document does not define:

final district heating design
final pipeline routing
legal heat sale contracts
civil engineering
building-side retrofit
final insulation material
municipal permitting
final economic payback
2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

Heat export priority condition:

Q_export is allocated after Q_WCT, Q_ADC, and Q_WENV.
Q_export is allocated before Q_buffer.

Fallback rule:

If heat export demand is unavailable:
    route available heat to thermal buffers.

If thermal buffers are full or unavailable:
    route remaining heat to thermal tail.
3. Required Inputs
3.1 SPHERE Heat Inputs
Symbol	Meaning	Unit
Q_captured	Heat captured by SPHERE thermal core	kW
Q_WCT	Heat routed to W-CT	kW
Q_ADC	Heat routed to A-DC	kW
Q_WENV	Heat routed to W-env-DC	kW
Q_export_available	Heat available for export after higher-priority modules	kW
Q_export_candidate	Heat selected for export before losses	kW
Q_export_interface_max	Maximum export interface heat transfer capacity	kW
3.2 External Demand Inputs
Symbol	Meaning	Unit
Q_export_demand	External heat demand at current time	kW
Q_export_demand_peak	Peak external heat demand	kW
Q_export_demand_min	Minimum external heat demand	kW
E_export_demand_day	Daily external heat demand	kWh/day
demand_availability_factor	Fraction of time external demand is available	0–1
T_required_supply	Required supply temperature for external user	°C
T_required_return	Expected return temperature from external user	°C
3.3 Export Loop Inputs
Symbol	Meaning	Unit
T_export_supply	Export loop supply temperature	°C
T_export_return	Export loop return temperature	°C
DeltaT_export	Export loop temperature difference	K
m_dot_export	Export loop mass flow	kg/s
V_dot_export	Export loop volume flow	m³/h
Cp_export_fluid	Specific heat of export fluid	kJ/kg·K
rho_export_fluid	Density of export fluid	kg/m³
eta_HX_export	Export heat exchanger efficiency	0–1
3.4 Distribution Loss Inputs
Symbol	Meaning	Unit
L_pipe	One-way pipe length to heat user	m
D_pipe_outer	Pipe outer diameter	m
N_pipes	Number of thermal pipes, usually supply + return	count
U_pipe	Overall heat loss coefficient of insulated pipe	W/m²·K
A_pipe_external	External heat loss area of pipe	m²
T_fluid_avg	Average fluid temperature in export loop	°C
T_ambient	Ambient temperature around pipe	°C
Q_loss_pipe	Heat loss during transfer	kW
eta_distribution	Distribution efficiency	0–1
3.5 Pump / Electric Inputs
Symbol	Meaning	Unit
DeltaP_export	Export loop pressure drop	Pa
eta_pump_export	Export pump efficiency	0–1
P_pump_export	Export pump electric power	kW
P_controls_export	Export control and sensor electric load	kW
P_export_electric	Total heat export electric load	kW
3.6 Output Variables
Symbol	Meaning	Unit
Q_export_delivered	Heat delivered to external user after losses	kW
Q_export_useful	Delivered heat that matches real external demand	kW
Q_export_rejected	Export heat not used by external demand	kW
E_export_delivered_day	Delivered heat energy per day	kWh/day
E_export_useful_day	Useful exported heat energy per day	kWh/day
export_useful_ratio	Useful export fraction	0–1
Q_export_to_buffer	Export heat rerouted to buffer	kW
Q_export_to_tail	Export heat rerouted to thermal tail	kW
4. Default Constants
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³
seconds_per_hour = 3600 s/h
hours_per_day = 24 h/day
pi = 3.14159

Default export fluid:

Cp_export_fluid = Cp_water
rho_export_fluid = rho_water

Site-specific fluid properties must replace defaults if glycol, brine, or another heat transfer fluid is used.

5. Available Heat for Export

After higher-priority modules:

Q_export_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV

Constraint:

Q_export_available >= 0

If calculated value is negative:

Q_export_available = 0

Candidate export heat:

Q_export_candidate =
min(
    Q_export_available,
    Q_export_demand,
    Q_export_interface_max
)

Constraints:

Q_export_candidate <= Q_export_available
Q_export_candidate <= Q_export_demand
Q_export_candidate <= Q_export_interface_max
6. Temperature Suitability

Temperature difference:

DeltaT_export =
T_export_supply - T_export_return

Constraint:

DeltaT_export > 0

Supply suitability condition:

T_export_supply >= T_required_supply + T_margin_supply

Return compatibility condition:

T_export_return <= T_required_return + T_margin_return

If temperature is insufficient:

heat_export_status = "temperature_not_suitable"
Q_export_useful = 0

Non-claim rule:

Low-grade heat must not be counted as useful export unless the external user can use that temperature level.
7. Export Loop Flow Rate

Mass flow required:

m_dot_export =
Q_export_candidate / (Cp_export_fluid × DeltaT_export)

Where:

Q_export_candidate in kW = kJ/s
Cp_export_fluid in kJ/kg·K
DeltaT_export in K
m_dot_export in kg/s

Volume flow:

V_dot_export_m3_s =
m_dot_export / rho_export_fluid
V_dot_export =
V_dot_export_m3_s × 3600

Where:

V_dot_export in m³/h

Constraint:

V_dot_export <= V_dot_export_max_allowed
8. Heat Exchanger Transfer

Heat after export heat exchanger:

Q_after_HX =
Q_export_candidate × eta_HX_export

Constraint:

0 < eta_HX_export <= 1
Q_after_HX <= Q_export_candidate

Heat exchanger loss:

Q_loss_HX =
Q_export_candidate - Q_after_HX
9. Pipe / Distribution Losses

Pipe external area:

A_pipe_external =
pi × D_pipe_outer × L_pipe × N_pipes

Average loop temperature:

T_fluid_avg =
(T_export_supply + T_export_return) / 2

Pipe heat loss:

Q_loss_pipe =
(U_pipe × A_pipe_external × (T_fluid_avg - T_ambient)) / 1000

Where:

U_pipe in W/m²·K
A_pipe_external in m²
temperature difference in K
Q_loss_pipe in kW

If:

T_fluid_avg <= T_ambient

Then:

Q_loss_pipe = 0

Alternative efficiency form:

eta_distribution =
1 - (Q_loss_pipe / Q_after_HX)

Constraint:

0 <= eta_distribution <= 1
Q_loss_pipe <= Q_after_HX
10. Delivered Heat
Q_export_delivered =
max(0, Q_after_HX - Q_loss_pipe)

Equivalent:

Q_export_delivered =
Q_export_candidate × eta_HX_export × eta_distribution

Constraint:

Q_export_delivered <= Q_export_candidate
11. Useful Exported Heat

Useful heat is limited by real demand:

Q_export_useful =
min(Q_export_delivered, Q_export_demand)

Rejected / unmatched delivered heat:

Q_export_rejected =
Q_export_delivered - Q_export_useful

Constraints:

Q_export_useful <= Q_export_delivered
Q_export_useful <= Q_export_demand
Q_export_rejected >= 0

Validity condition:

Q_export_useful counts only if it replaces existing heat demand.

Non-claim rule:

Exporting heat to an unused sink is not useful heat reuse.
It is external heat rejection.
12. Daily Export Energy

For steady operation:

E_export_delivered_day =
Q_export_delivered × operating_hours_export
E_export_useful_day =
Q_export_useful × operating_hours_export

For time-series operation:

E_export_delivered_day =
sum(Q_export_delivered_t × Delta_t)
E_export_useful_day =
sum(min(Q_export_delivered_t, Q_export_demand_t) × Delta_t)

Useful ratio:

export_useful_ratio =
E_export_useful_day / E_export_delivered_day

Constraint:

0 <= export_useful_ratio <= 1
13. Pump Power

Export loop pump power:

P_pump_export =
(DeltaP_export × V_dot_export_m3_s) / eta_pump_export / 1000

Where:

DeltaP_export in Pa
V_dot_export_m3_s in m³/s
P_pump_export in kW

Total export electric load:

P_export_electric =
P_pump_export
+ P_controls_export

Daily electric energy:

E_export_electric_day =
P_export_electric × operating_hours_export

Constraint:

0 < eta_pump_export <= 1
P_export_electric must be included in SPHERE system electric load.
14. Net Useful Export Metric

Thermal delivery is not directly subtractable from electric pump power without valuation.

Report separately:

E_export_useful_day_kWh_th
E_export_electric_day_kWh_el

Optional ratio:

thermal_export_per_electric_input =
E_export_useful_day / E_export_electric_day

Constraint:

E_export_electric_day > 0

Non-claim rule:

Do not present thermal kWh and electric kWh as equivalent without explicit conversion basis.
15. Replacement Logic

Exported heat is useful only when replacing another heat source.

Replacement condition:

if external_user_has_existing_heat_demand == true
and T_export_supply >= T_required_supply
and Q_export_demand > 0:
    exported_heat_status = "useful_replacement"
else:
    exported_heat_status = "external_rejection"

Avoided heat input:

E_heat_replaced_day =
E_export_useful_day × replacement_factor

Where:

replacement_factor in 0–1

Constraint:

0 <= replacement_factor <= 1

Non-claim rule:

Environmental or economic benefit requires defining the replaced heat source.
16. Reference Case A: 1 MW Data Center
16.1 Declared Inputs
P_IT = 1,000 kW

Q_captured = 630 kW
Q_WCT = 200 kW
Q_ADC = 150 kW
Q_WENV = 100 kW

Q_export_demand = 120 kW
Q_export_interface_max = 150 kW

T_export_supply = 55°C
T_export_return = 35°C
T_required_supply = 50°C
T_margin_supply = 0 K

Cp_export_fluid = 4.186 kJ/kg·K
rho_export_fluid = 1000 kg/m³

eta_HX_export = 0.95

L_pipe = 100 m
D_pipe_outer = 0.10 m
N_pipes = 2
U_pipe = 0.35 W/m²·K
T_ambient = 20°C

DeltaP_export = 80,000 Pa
eta_pump_export = 0.65
P_controls_export = 0.5 kW

operating_hours_export = 12 h/day
replacement_factor = 1.0
16.2 Available Export Heat
Q_export_available =
630 - 200 - 150 - 100

Q_export_available = 180 kW
Q_export_candidate =
min(180, 120, 150)

Q_export_candidate = 120 kW
16.3 Temperature Suitability
DeltaT_export =
55 - 35

DeltaT_export = 20 K
T_export_supply = 55°C
T_required_supply = 50°C

temperature_suitable = true
16.4 Flow Rate
m_dot_export =
120 / (4.186 × 20)

m_dot_export = 1.433 kg/s
V_dot_export_m3_s =
1.433 / 1000

V_dot_export_m3_s = 0.001433 m³/s
V_dot_export =
0.001433 × 3600

V_dot_export = 5.16 m³/h
16.5 Heat Exchanger Output
Q_after_HX =
120 × 0.95

Q_after_HX = 114 kW
Q_loss_HX =
120 - 114

Q_loss_HX = 6 kW
16.6 Pipe Loss
A_pipe_external =
3.14159 × 0.10 × 100 × 2

A_pipe_external = 62.83 m²
T_fluid_avg =
(55 + 35) / 2

T_fluid_avg = 45°C
Q_loss_pipe =
(0.35 × 62.83 × (45 - 20)) / 1000

Q_loss_pipe =
549.76 / 1000

Q_loss_pipe = 0.55 kW
16.7 Delivered and Useful Heat
Q_export_delivered =
114 - 0.55

Q_export_delivered = 113.45 kW
Q_export_useful =
min(113.45, 120)

Q_export_useful = 113.45 kW
Q_export_rejected =
113.45 - 113.45

Q_export_rejected = 0 kW

Daily useful export:

E_export_useful_day =
113.45 × 12

E_export_useful_day = 1361.4 kWh_th/day
16.8 Pump Power
P_pump_export =
(80,000 × 0.001433) / 0.65 / 1000

P_pump_export = 0.176 kW
P_export_electric =
0.176 + 0.5

P_export_electric = 0.676 kW
E_export_electric_day =
0.676 × 12

E_export_electric_day = 8.11 kWh_el/day

Thermal export per electric input:

thermal_export_per_electric_input =
1361.4 / 8.11

thermal_export_per_electric_input = 167.9 kWh_th/kWh_el

Interpretation:

Under declared assumptions, export pumping energy is small compared with delivered thermal energy.
This depends strongly on pipe distance, pressure drop, temperature level, and demand availability.
17. Reference Case B: 10 MW Data Center
17.1 Declared Inputs

Scale from Case A:

P_IT = 10,000 kW

Q_captured = 6,300 kW
Q_WCT = 2,000 kW
Q_ADC = 1,500 kW
Q_WENV = 1,000 kW

Q_export_demand = 1,200 kW
Q_export_interface_max = 1,500 kW

T_export_supply = 55°C
T_export_return = 35°C
T_required_supply = 50°C
T_margin_supply = 0 K

Cp_export_fluid = 4.186 kJ/kg·K
rho_export_fluid = 1000 kg/m³

eta_HX_export = 0.95

L_pipe = 300 m
D_pipe_outer = 0.25 m
N_pipes = 2
U_pipe = 0.30 W/m²·K
T_ambient = 20°C

DeltaP_export = 120,000 Pa
eta_pump_export = 0.70
P_controls_export = 2.0 kW

operating_hours_export = 12 h/day
replacement_factor = 1.0
17.2 Available Export Heat
Q_export_available =
6300 - 2000 - 1500 - 1000

Q_export_available = 1800 kW
Q_export_candidate =
min(1800, 1200, 1500)

Q_export_candidate = 1200 kW
17.3 Temperature Suitability
DeltaT_export =
55 - 35

DeltaT_export = 20 K
T_export_supply = 55°C
T_required_supply = 50°C

temperature_suitable = true
17.4 Flow Rate
m_dot_export =
1200 / (4.186 × 20)

m_dot_export = 14.33 kg/s
V_dot_export_m3_s =
14.33 / 1000

V_dot_export_m3_s = 0.01433 m³/s
V_dot_export =
0.01433 × 3600

V_dot_export = 51.6 m³/h
17.5 Heat Exchanger Output
Q_after_HX =
1200 × 0.95

Q_after_HX = 1140 kW
Q_loss_HX =
1200 - 1140

Q_loss_HX = 60 kW
17.6 Pipe Loss
A_pipe_external =
3.14159 × 0.25 × 300 × 2

A_pipe_external = 471.24 m²
T_fluid_avg =
(55 + 35) / 2

T_fluid_avg = 45°C
Q_loss_pipe =
(0.30 × 471.24 × (45 - 20)) / 1000

Q_loss_pipe =
3534.3 / 1000

Q_loss_pipe = 3.53 kW
17.7 Delivered and Useful Heat
Q_export_delivered =
1140 - 3.53

Q_export_delivered = 1136.47 kW
Q_export_useful =
min(1136.47, 1200)

Q_export_useful = 1136.47 kW
Q_export_rejected =
1136.47 - 1136.47

Q_export_rejected = 0 kW

Daily useful export:

E_export_useful_day =
1136.47 × 12

E_export_useful_day = 13,637.6 kWh_th/day
17.8 Pump Power
P_pump_export =
(120,000 × 0.01433) / 0.70 / 1000

P_pump_export = 2.46 kW
P_export_electric =
2.46 + 2.0

P_export_electric = 4.46 kW
E_export_electric_day =
4.46 × 12

E_export_electric_day = 53.5 kWh_el/day

Thermal export per electric input:

thermal_export_per_electric_input =
13,637.6 / 53.5

thermal_export_per_electric_input = 254.9 kWh_th/kWh_el

Interpretation:

Under declared assumptions, heat export is technically efficient at this distance and pressure drop.
Feasibility still depends on stable external demand and suitable temperature level.
18. Demand Mismatch Example

If delivered heat exceeds external demand:

Q_export_delivered = 500 kW
Q_export_demand = 300 kW

Then:

Q_export_useful =
min(500, 300)

Q_export_useful = 300 kW
Q_export_rejected =
500 - 300

Q_export_rejected = 200 kW

Interpretation:

Only 300 kW counts as useful heat export.
The remaining 200 kW must be routed to buffer, reduced at source, or treated as external heat rejection.
19. Temperature Mismatch Example

If:

T_export_supply = 40°C
T_required_supply = 60°C

Then:

temperature_suitable = false
Q_export_useful = 0

Unless:

heat pump or temperature lift module is added and modeled separately.

Non-claim rule:

Do not count low-temperature heat as useful for a high-temperature user without modeling temperature lift energy.
20. Process Output Requirements

Heat export calculation must output:

Q_export_available
Q_export_demand
Q_export_interface_max
Q_export_candidate
T_export_supply
T_export_return
DeltaT_export
T_required_supply
temperature_suitable
m_dot_export
V_dot_export
eta_HX_export
Q_after_HX
Q_loss_HX
L_pipe
D_pipe_outer
A_pipe_external
U_pipe
Q_loss_pipe
eta_distribution
Q_export_delivered
Q_export_useful
Q_export_rejected
E_export_delivered_day
E_export_useful_day
export_useful_ratio
DeltaP_export
P_pump_export
P_export_electric
E_export_electric_day
thermal_export_per_electric_input
Q_export_to_buffer
Q_export_to_tail
21. Control Logic
INPUT:
    Q_captured
    Q_WCT
    Q_ADC
    Q_WENV
    external_heat_demand
    T_export_supply
    T_required_supply
    buffer_status
    export_loop_status

PROCESS:
    calculate Q_export_available

    check external demand availability

    check temperature suitability

    calculate Q_export_candidate

    calculate export flow rate

    calculate heat exchanger transfer

    calculate pipe losses

    calculate delivered heat

    calculate useful heat based on real demand

    calculate pump power

IF external demand available AND temperature suitable:
    export heat up to demand and interface limit

ELSE IF buffer has capacity:
    route heat to thermal buffer

ELSE:
    route heat to thermal tail

IF delivered heat exceeds demand:
    count only matched demand as useful
    route excess to buffer or tail

OUTPUT:
    useful exported heat
    rejected / unmatched heat
    electric cost of export
    updated buffer or tail routing
22. Failure / Bypass Conditions
IF heat_export_fault == true:
    isolate heat export loop
    preserve SPHERE internal module priority
    route available export heat to thermal buffer or thermal tail
IF external_user_unavailable == true:
    stop export
    route heat to buffer or tail
IF T_export_supply < T_required_supply:
    stop useful export claim
    route heat to lower-temperature user, buffer, or tail
IF DeltaP_export > DeltaP_max_allowed:
    reduce flow
    reduce export heat
    or isolate export loop
IF leak_detected == true:
    isolate export loop
    preserve data center cooling safety
IF Q_loss_pipe > Q_loss_pipe_max_allowed:
    reduce export distance
    improve insulation
    lower export claim
    or cancel export path
23. Engineering Constraints
Q_export_available >= 0

Q_export_candidate <= Q_export_available

Q_export_candidate <= Q_export_demand

Q_export_candidate <= Q_export_interface_max

DeltaT_export > 0

T_export_supply must satisfy external demand temperature.

Q_export_delivered <= Q_export_candidate

Q_export_useful <= Q_export_delivered

Q_export_useful <= Q_export_demand

Q_export_rejected >= 0

0 < eta_HX_export <= 1

0 <= eta_distribution <= 1

0 < eta_pump_export <= 1

P_export_electric must be included in SPHERE system electric load.

Exported heat counts as useful only if it replaces real heat demand.

Heat export must be bypassable.

Heat export must not compromise primary data center cooling.

Heat export must not reduce heat available to higher-priority SPHERE modules.
24. AI Validation Questions

An AI analysis system should verify:

1. Is Q_export_available calculated only after W-CT, A-DC, and W-env-DC?

2. Is Q_export_candidate limited by available heat, external demand, and interface capacity?

3. Is the external heat user defined?

4. Is external demand real, time-matched, and quantified?

5. Is T_export_supply sufficient for the external user?

6. Is DeltaT_export positive and realistic?

7. Is mass flow calculated from Q, Cp, and DeltaT?

8. Is pipe length included?

9. Are pipe heat losses calculated?

10. Is heat exchanger efficiency included?

11. Is delivered heat lower than candidate heat after losses?

12. Is useful exported heat limited by external demand?

13. Is unmatched delivered heat separated from useful export?

14. Is pump power calculated from pressure drop and flow?

15. Is export electric load included in system load?

16. Are thermal kWh and electric kWh reported separately?

17. Is exported heat counted only when replacing existing heat demand?

18. Is fallback routing defined if export user is unavailable?

19. Is heat export avoided as a zero-thermal-tail claim?

20. Which assumptions dominate feasibility: distance, temperature, external demand, DeltaT, pipe losses, or pump power?
25. Minimum Site Data Required
available Q_export profile
hourly SPHERE module heat allocation
external heat user identity
external heat demand profile
required supply temperature
expected return temperature
distance to heat user
pipe route
allowed pipe diameter
insulation constraints
ambient temperature profile
soil / air installation environment
export heat exchanger specification
allowed pressure drop
pump efficiency
export loop fluid type
freeze protection requirements
maintenance access
heat metering requirements
BMS / SCADA integration points
fallback heat rejection capacity
thermal buffer capacity
legal / contractual export conditions
existing heat source being replaced
replacement_factor
26. Non-Claim Rules

Not allowed without site-specific proof:

heat elimination
zero thermal tail
useful heat export without external demand
district heating compatibility without temperature match
free heat delivery
lossless heat transfer
unlimited export distance
no pump energy penalty
guaranteed environmental benefit
guaranteed economic payback
exported heat counted as useful when dumped to atmosphere

Allowed claim format:

Under declared assumptions:
SPHERE-DC has X kW available for heat export.
Y kW can be transferred to the external user interface.
Z kW is delivered after heat exchanger and pipe losses.
A kW counts as useful heat because it matches real external demand.
B kW is unmatched and must route to buffer or thermal tail.
Export requires C kW electric pump/control load.
27. Summary Formula Block
Q_export_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV

Q_export_candidate =
min(
    Q_export_available,
    Q_export_demand,
    Q_export_interface_max
)

DeltaT_export =
T_export_supply - T_export_return

m_dot_export =
Q_export_candidate / (Cp_export_fluid × DeltaT_export)

V_dot_export_m3_s =
m_dot_export / rho_export_fluid

V_dot_export =
V_dot_export_m3_s × 3600

Q_after_HX =
Q_export_candidate × eta_HX_export

Q_loss_HX =
Q_export_candidate - Q_after_HX

A_pipe_external =
pi × D_pipe_outer × L_pipe × N_pipes

T_fluid_avg =
(T_export_supply + T_export_return) / 2

Q_loss_pipe =
(U_pipe × A_pipe_external × (T_fluid_avg - T_ambient)) / 1000

eta_distribution =
1 - (Q_loss_pipe / Q_after_HX)

Q_export_delivered =
max(0, Q_after_HX - Q_loss_pipe)

Q_export_delivered =
Q_export_candidate × eta_HX_export × eta_distribution

Q_export_useful =
min(Q_export_delivered, Q_export_demand)

Q_export_rejected =
Q_export_delivered - Q_export_useful

E_export_delivered_day =
Q_export_delivered × operating_hours_export

E_export_useful_day =
Q_export_useful × operating_hours_export

export_useful_ratio =
E_export_useful_day / E_export_delivered_day

P_pump_export =
(DeltaP_export × V_dot_export_m3_s) / eta_pump_export / 1000

P_export_electric =
P_pump_export
+ P_controls_export

E_export_electric_day =
P_export_electric × operating_hours_export

thermal_export_per_electric_input =
E_export_useful_day / E_export_electric_day

E_heat_replaced_day =
E_export_useful_day × replacement_factor