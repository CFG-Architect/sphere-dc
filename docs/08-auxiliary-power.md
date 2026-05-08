# auxiliary-power.md

# SPHERE-DC Technical Document 08: Auxiliary Power

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
Auxiliary Power = limited internal energy recovery layer for SPHERE-DC support loads

Concept baseline:

SPHERE-DC may convert a small fraction of available thermal energy into auxiliary electricity.

Auxiliary power may support:
- pumps
- fans
- sensors
- control systems
- valves
- monitoring
- automation
- small robotic or maintenance subsystems

Auxiliary power must not be modeled as powering the data center itself.
Auxiliary power must not be claimed as self-powering the full SPHERE-DC system unless net balance is proven.
```

1. Scope

This document defines calculation logic for:

available heat for auxiliary conversion
temperature-level suitability
maximum theoretical conversion efficiency
practical conversion efficiency
gross auxiliary electric output
internal SPHERE electric loads
net auxiliary power balance
daily auxiliary energy balance
pump loads
fan loads
control loads
conversion waste heat
failure and bypass conditions

This document does not define:

final ORC design
final thermoelectric generator selection
final turbine selection
final working fluid
power electronics design
grid interconnection
battery design
electrical safety certification
economic payback
2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

Auxiliary power heat source rule:

Auxiliary power must not steal heat required by higher-priority SPHERE modules.

Q_aux_available may be taken from:
- residual captured heat after priority modules
- heat buffer discharge
- dedicated small heat fraction
- temperature levels unsuitable for higher-value process use

Priority condition:

Auxiliary power is a support function.
It is not a primary heat-use priority unless explicitly declared in the system allocation.
3. Required Inputs
3.1 Heat Inputs
Symbol	Meaning	Unit
Q_total	Total heat load inside selected boundary	kW
Q_captured	Heat captured by SPHERE thermal core	kW
Q_WCT	Heat routed to W-CT	kW
Q_ADC	Heat routed to A-DC	kW
Q_WENV	Heat routed to W-env-DC	kW
Q_export	Heat routed to heat export	kW
Q_buffer	Heat routed to thermal buffer	kW
Q_aux_available	Heat available for auxiliary conversion	kW
Q_aux_used	Heat actually routed to auxiliary conversion	kW
Q_aux_rejected	Residual heat after auxiliary conversion	kW
3.2 Temperature Inputs
Symbol	Meaning	Unit
T_hot_aux_C	Hot-side temperature for conversion	°C
T_cold_aux_C	Cold-side / sink temperature	°C
T_hot_aux_K	Hot-side temperature	K
T_cold_aux_K	Cold-side temperature	K
DeltaT_aux	Temperature difference	K
3.3 Conversion Inputs
Symbol	Meaning	Unit
eta_Carnot_aux	Theoretical maximum efficiency	0–1
eta_practical_fraction	Fraction of Carnot achievable by selected technology	0–1
eta_aux_conversion	Practical heat-to-electric efficiency	0–1
P_aux_gross	Gross auxiliary electric output	kW
P_aux_net	Net auxiliary electric balance	kW
E_aux_gross_day	Gross daily auxiliary electric energy	kWh/day
E_aux_net_day	Net daily auxiliary electric energy	kWh/day
operating_hours_aux	Daily operation time	h/day
3.4 Internal Load Inputs
Symbol	Meaning	Unit
P_pumps_total	Total pump electric load	kW
P_fans_total	Total fan electric load	kW
P_controls_total	Control / sensor / automation load	kW
P_valves_total	Valve / actuator load	kW
P_treatment_total	Water / air treatment electric load	kW
P_power_electronics_loss	Conversion electronics loss	kW
P_aux_internal_load	Subset of SPHERE-DC internal electric loads intended to be offset by auxiliary power; must not be double-counted against the same pumps, fans, controls, treatment loads, or power electronics losses already listed elsewhere	kW
P_backup_required	Required backup electric power if auxiliary fails	kW
3.5 Storage / Power Conditioning Inputs
Symbol	Meaning	Unit
eta_power_electronics	Power electronics efficiency	0–1
eta_storage_charge	Battery / storage charge efficiency	0–1
eta_storage_discharge	Battery / storage discharge efficiency	0–1
E_storage_aux	Auxiliary electrical storage capacity	kWh
SOC_aux_storage	Auxiliary storage state of charge	0–1
4. Default Constants
K_offset = 273.15
hours_per_day = 24

Temperature conversion:

T_hot_aux_K = T_hot_aux_C + 273.15
T_cold_aux_K = T_cold_aux_C + 273.15
5. Available Heat for Auxiliary Conversion

If auxiliary heat comes after all priority heat uses:

Q_aux_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export
- Q_buffer

Constraint:

Q_aux_available >= 0

If calculated value is negative:

Q_aux_available = 0

If auxiliary conversion uses a declared dedicated heat fraction:

Q_aux_available =
Q_captured × f_aux_heat

Constraint:

0 <= f_aux_heat <= 1

Dedicated heat fraction must not reduce required heat for higher-priority modules unless explicitly modeled.

6. Temperature Suitability

Temperature difference:

DeltaT_aux =
T_hot_aux_C - T_cold_aux_C

Constraint:

DeltaT_aux > 0

Absolute temperature conversion:

T_hot_aux_K =
T_hot_aux_C + 273.15

T_cold_aux_K =
T_cold_aux_C + 273.15

Constraint:

T_hot_aux_K > T_cold_aux_K > 0

If:

DeltaT_aux <= 0

Then:

P_aux_gross = 0
auxiliary_conversion_status = "not_possible"
7. Theoretical Efficiency Limit

Carnot efficiency:

eta_Carnot_aux =
1 - (T_cold_aux_K / T_hot_aux_K)

Constraint:

0 < eta_Carnot_aux < 1

Practical conversion efficiency:

eta_aux_conversion =
eta_Carnot_aux × eta_practical_fraction

Constraint:

0 <= eta_practical_fraction <= 1
0 <= eta_aux_conversion < eta_Carnot_aux

Practical note:

For low-grade heat, eta_aux_conversion is usually small.
Do not assume high heat-to-electric efficiency without technology-specific evidence.
8. Gross Auxiliary Power
Q_aux_used =
min(Q_aux_available, Q_aux_conversion_capacity)

Gross output before power electronics losses:

P_aux_gross_raw =
Q_aux_used × eta_aux_conversion

Gross usable electric output after power electronics:

P_aux_gross =
P_aux_gross_raw × eta_power_electronics

Constraints:

Q_aux_used <= Q_aux_available
P_aux_gross <= Q_aux_used
0 < eta_power_electronics <= 1
9. Residual Heat After Conversion

Heat not converted to electricity:

Q_aux_rejected =
Q_aux_used - P_aux_gross_raw

If including power electronics losses as heat:

Q_power_electronics_heat =
P_aux_gross_raw - P_aux_gross

Total residual heat from auxiliary conversion:

Q_aux_residual_total =
Q_aux_rejected + Q_power_electronics_heat

Constraint:

Q_aux_residual_total >= 0

Routing rule:

Q_aux_residual_total must route to:
- W-CT
- A-DC
- W-env-DC
- heat export
- thermal buffer
- thermal tail

Non-claim rule:

Auxiliary conversion reduces a portion of heat to electricity, but most low-grade heat remains as residual heat.
10. Internal Load Model

Total internal electric load assigned to auxiliary power:

P_aux_internal_load =
P_pumps_total
+ P_fans_total
+ P_controls_total
+ P_valves_total
+ P_treatment_total
+ P_power_electronics_loss

Double-counting rule:

P_aux_internal_load is not an additional electric load category.

P_aux_internal_load is the subset of already declared SPHERE-DC internal electric loads that auxiliary power attempts to offset.

A pump, fan, control system, valve, treatment unit, or power electronics loss must not be counted once inside P_aux_internal_load and again as a separate additional load in the same integrated electric balance.

Net auxiliary balance:

P_aux_net =
P_aux_gross - P_aux_internal_load

Interpretation:

If P_aux_net > 0:
    auxiliary system produces surplus support electricity.

If P_aux_net = 0:
    auxiliary system covers assigned internal loads exactly.

If P_aux_net < 0:
    auxiliary system does not cover assigned internal loads.
    External or backup electricity is required.

Constraint:

P_aux_internal_load must be explicitly declared.
11. Daily Energy Balance

Gross daily auxiliary energy:

E_aux_gross_day =
P_aux_gross × operating_hours_aux

Internal daily load:

E_aux_internal_load_day =
P_aux_internal_load × operating_hours_aux

Net daily auxiliary energy:

E_aux_net_day =
E_aux_gross_day - E_aux_internal_load_day

Residual daily heat from auxiliary conversion:

E_aux_residual_heat_day =
Q_aux_residual_total × operating_hours_aux

Constraint:

operating_hours_aux <= 24
12. Pump Load Submodel

For each liquid loop i:

P_pump_i =
(DeltaP_i × V_dot_i_m3_s) / eta_pump_i / 1000

Total pump load:

P_pumps_total =
sum(P_pump_i)

Constraints:

DeltaP_i >= 0
V_dot_i_m3_s >= 0
0 < eta_pump_i <= 1
13. Fan Load Submodel

For each air loop j:

P_fan_j =
(DeltaP_air_j × V_dot_air_j_m3_s) / eta_fan_j / 1000

Total fan load:

P_fans_total =
sum(P_fan_j)

Constraints:

DeltaP_air_j >= 0
V_dot_air_j_m3_s >= 0
0 < eta_fan_j <= 1
14. Power Electronics and Storage

Power electronics output:

P_after_electronics =
P_aux_gross_raw × eta_power_electronics

If charging storage:

P_to_storage =
P_after_electronics - P_direct_load

Storage charge condition:

if P_to_storage > 0 and SOC_aux_storage < SOC_max:
    charge storage

Storage state update:

SOC_aux_storage_next =
SOC_aux_storage
+ (P_to_storage × eta_storage_charge × Delta_t) / E_storage_aux
- (P_from_storage × Delta_t) / (E_storage_aux × eta_storage_discharge)

Clamp:

SOC_aux_storage_next =
min(SOC_max, max(SOC_min, SOC_aux_storage_next))

Constraints:

0 <= SOC_min < SOC_max <= 1
0 <= SOC_aux_storage <= 1
0 < eta_storage_charge <= 1
0 < eta_storage_discharge <= 1
15. Technology Classes

Allowed generic technology classes:

ORC = Organic Rankine Cycle
TEG = Thermoelectric Generator
microturbine = only if temperature level is suitable
Stirling = only if temperature level is suitable
TPV = only for high-temperature emitters, not baseline low-grade heat

Technology-specific efficiency must be declared.

Generic practical efficiency model:

eta_aux_conversion =
eta_Carnot_aux × eta_practical_fraction

Typical early-concept assumption range:

eta_practical_fraction = 0.05 to 0.30 of Carnot

Non-claim rule:

Do not use high-temperature conversion efficiencies for low-temperature data center heat.
16. Reference Case A: 1 MW Data Center
16.1 Declared Inputs
P_IT = 1,000 kW

Q_captured = 630 kW
Q_WCT = 200 kW
Q_ADC = 150 kW
Q_WENV = 100 kW
Q_export = 100 kW
Q_buffer = 50 kW

Q_aux_conversion_capacity = 100 kW

T_hot_aux_C = 80°C
T_cold_aux_C = 30°C

eta_practical_fraction = 0.15
eta_power_electronics = 0.95

P_pumps_total = 3.0 kW
P_fans_total = 2.0 kW
P_controls_total = 0.8 kW
P_valves_total = 0.2 kW
P_treatment_total = 1.5 kW
P_power_electronics_loss = 0 kW

operating_hours_aux = 24 h/day
16.2 Available Heat
Q_aux_available =
630 - 200 - 150 - 100 - 100 - 50

Q_aux_available = 30 kW
Q_aux_used =
min(30, 100)

Q_aux_used = 30 kW
16.3 Carnot Limit
T_hot_aux_K =
80 + 273.15

T_hot_aux_K = 353.15 K
T_cold_aux_K =
30 + 273.15

T_cold_aux_K = 303.15 K
eta_Carnot_aux =
1 - (303.15 / 353.15)

eta_Carnot_aux = 0.1416

Practical efficiency:

eta_aux_conversion =
0.1416 × 0.15

eta_aux_conversion = 0.0212

Result:

Practical conversion efficiency = 2.12%
16.4 Gross Auxiliary Power
P_aux_gross_raw =
30 × 0.0212

P_aux_gross_raw = 0.636 kW
P_aux_gross =
0.636 × 0.95

P_aux_gross = 0.604 kW
16.5 Internal Load
P_aux_internal_load =
3.0 + 2.0 + 0.8 + 0.2 + 1.5 + 0

P_aux_internal_load = 7.5 kW

Net:

P_aux_net =
0.604 - 7.5

P_aux_net = -6.896 kW

Result:

Under declared assumptions, auxiliary conversion does not cover assigned SPHERE internal loads.
External electricity required = 6.896 kW.
16.6 Daily Energy
E_aux_gross_day =
0.604 × 24

E_aux_gross_day = 14.5 kWh/day
E_aux_internal_load_day =
7.5 × 24

E_aux_internal_load_day = 180 kWh/day
E_aux_net_day =
14.5 - 180

E_aux_net_day = -165.5 kWh/day
16.7 Residual Heat
Q_aux_rejected =
30 - 0.636

Q_aux_rejected = 29.364 kW
Q_power_electronics_heat =
0.636 - 0.604

Q_power_electronics_heat = 0.032 kW
Q_aux_residual_total =
29.364 + 0.032

Q_aux_residual_total = 29.396 kW

Interpretation:

Most low-grade auxiliary conversion heat remains as residual heat.
This residual must be routed to another process, buffer, or thermal tail.
17. Reference Case B: 10 MW Data Center
17.1 Declared Inputs

Scale from Case A but allow larger residual heat fraction:

P_IT = 10,000 kW

Q_captured = 6,300 kW
Q_WCT = 2,000 kW
Q_ADC = 1,500 kW
Q_WENV = 1,000 kW
Q_export = 1,000 kW
Q_buffer = 500 kW

Q_aux_conversion_capacity = 500 kW

T_hot_aux_C = 80°C
T_cold_aux_C = 30°C

eta_practical_fraction = 0.15
eta_power_electronics = 0.95

P_pumps_total = 20 kW
P_fans_total = 15 kW
P_controls_total = 3 kW
P_valves_total = 2 kW
P_treatment_total = 10 kW
P_power_electronics_loss = 0 kW

operating_hours_aux = 24 h/day
17.2 Available Heat
Q_aux_available =
6300 - 2000 - 1500 - 1000 - 1000 - 500

Q_aux_available = 300 kW
Q_aux_used =
min(300, 500)

Q_aux_used = 300 kW
17.3 Carnot Limit
T_hot_aux_K = 353.15 K
T_cold_aux_K = 303.15 K
eta_Carnot_aux =
1 - (303.15 / 353.15)

eta_Carnot_aux = 0.1416
eta_aux_conversion =
0.1416 × 0.15

eta_aux_conversion = 0.0212

Result:

Practical conversion efficiency = 2.12%
17.4 Gross Auxiliary Power
P_aux_gross_raw =
300 × 0.0212

P_aux_gross_raw = 6.36 kW
P_aux_gross =
6.36 × 0.95

P_aux_gross = 6.04 kW
17.5 Internal Load
P_aux_internal_load =
20 + 15 + 3 + 2 + 10 + 0

P_aux_internal_load = 50 kW

Net:

P_aux_net =
6.04 - 50

P_aux_net = -43.96 kW

Result:

Under declared low-grade heat assumptions, auxiliary conversion does not cover assigned SPHERE internal loads.
External electricity required = 43.96 kW.
17.6 Daily Energy
E_aux_gross_day =
6.04 × 24

E_aux_gross_day = 145 kWh/day
E_aux_internal_load_day =
50 × 24

E_aux_internal_load_day = 1200 kWh/day
E_aux_net_day =
145 - 1200

E_aux_net_day = -1055 kWh/day
17.7 Residual Heat
Q_aux_rejected =
300 - 6.36

Q_aux_rejected = 293.64 kW
Q_power_electronics_heat =
6.36 - 6.04

Q_power_electronics_heat = 0.32 kW
Q_aux_residual_total =
293.64 + 0.32

Q_aux_residual_total = 293.96 kW

Interpretation:

Low-grade heat-to-electric conversion produces limited support power.
Most heat remains thermal and must still be routed.
18. Higher Temperature Sensitivity

Assume:

T_hot_aux_C = 120°C
T_cold_aux_C = 30°C
eta_practical_fraction = 0.20
Q_aux_used = 300 kW
eta_power_electronics = 0.95

Convert:

T_hot_aux_K = 393.15 K
T_cold_aux_K = 303.15 K

Carnot:

eta_Carnot_aux =
1 - (303.15 / 393.15)

eta_Carnot_aux = 0.2289

Practical:

eta_aux_conversion =
0.2289 × 0.20

eta_aux_conversion = 0.0458

Gross:

P_aux_gross_raw =
300 × 0.0458

P_aux_gross_raw = 13.74 kW

After electronics:

P_aux_gross =
13.74 × 0.95

P_aux_gross = 13.05 kW

Interpretation:

Higher hot-side temperature improves auxiliary conversion.
However, 120°C heat is not typical low-grade server heat and must be justified by system architecture.
19. Support Coverage Ratio

Auxiliary coverage ratio:

auxiliary_coverage_ratio =
P_aux_gross / P_aux_internal_load

Constraint:

P_aux_internal_load > 0

If:

auxiliary_coverage_ratio >= 1

Then:

assigned internal loads are covered under declared assumptions

If:

auxiliary_coverage_ratio < 1

Then:

external or backup electricity is required

Reference case values:

1 MW case:
auxiliary_coverage_ratio = 0.604 / 7.5 = 0.081

10 MW case:
auxiliary_coverage_ratio = 6.04 / 50 = 0.121
20. Process Output Requirements

Auxiliary power calculation must output:

Q_aux_available
Q_aux_used
Q_aux_conversion_capacity
T_hot_aux_C
T_cold_aux_C
T_hot_aux_K
T_cold_aux_K
DeltaT_aux
eta_Carnot_aux
eta_practical_fraction
eta_aux_conversion
eta_power_electronics
P_aux_gross_raw
P_aux_gross
Q_aux_rejected
Q_power_electronics_heat
Q_aux_residual_total
P_pumps_total
P_fans_total
P_controls_total
P_valves_total
P_treatment_total
P_aux_internal_load
P_aux_net
E_aux_gross_day
E_aux_internal_load_day
E_aux_net_day
auxiliary_coverage_ratio
P_backup_required
21. Control Logic
INPUT:
    Q_aux_available
    T_hot_aux_C
    T_cold_aux_C
    Q_aux_conversion_capacity
    P_aux_internal_load
    module_priority_status
    backup_power_status

PROCESS:
    verify higher-priority modules are not heat-starved
    calculate DeltaT_aux
    calculate eta_Carnot_aux
    select eta_practical_fraction
    calculate eta_aux_conversion
    calculate Q_aux_used
    calculate P_aux_gross_raw
    apply power electronics efficiency
    calculate P_aux_gross
    calculate internal electric load
    calculate P_aux_net
    calculate residual heat
    route residual heat
    update backup power requirement

IF P_aux_net >= 0:
    route surplus to auxiliary storage or secondary support loads

ELSE:
    request external / backup electricity equal to abs(P_aux_net)

IF DeltaT_aux <= minimum_conversion_DeltaT:
    disable auxiliary conversion
    route heat to buffer, export, or tail

IF higher_priority_module_requires_heat:
    reduce or disable Q_aux_used

OUTPUT:
    auxiliary gross power
    auxiliary net balance
    residual heat
    backup requirement
22. Failure / Bypass Conditions
IF auxiliary_power_fault == true:
    isolate conversion module
    switch SPHERE support loads to external / backup electricity
    route Q_aux_available to buffer, export, or thermal tail
IF T_hot_aux_C <= T_cold_aux_C:
    disable conversion
    route heat to fallback path
IF P_aux_gross < critical_control_load:
    use external / backup electricity for controls
IF power_electronics_fault == true:
    disconnect auxiliary generator
    route thermal input to bypass
IF storage_full == true:
    stop charging auxiliary storage
    route surplus electricity to secondary loads or dump load
IF storage_empty == true and P_aux_net < 0:
    use external / backup electricity
23. Engineering Constraints
Q_aux_used <= Q_aux_available

Q_aux_available must not reduce heat required by W-CT, A-DC, W-env-DC, or committed heat export.

T_hot_aux_K > T_cold_aux_K

DeltaT_aux > 0 for conversion.

eta_aux_conversion < eta_Carnot_aux

P_aux_gross <= Q_aux_used

P_aux_internal_load must be declared.

P_aux_net must be reported.

Q_aux_residual_total must be routed.

Auxiliary power must not be claimed as data center power.

Auxiliary power must not be claimed as full SPHERE self-power unless P_aux_net >= 0 for all required operating modes.

Backup power must be available for critical controls.
24. AI Validation Questions

An AI analysis system should verify:

1. Is Q_aux_available calculated after higher-priority heat allocations?

2. Does auxiliary conversion steal heat from W-CT, A-DC, W-env-DC, or committed heat export?

3. Are hot-side and cold-side temperatures declared?

4. Are Celsius temperatures converted to Kelvin before Carnot calculation?

5. Is eta_Carnot_aux calculated correctly?

6. Is eta_aux_conversion lower than eta_Carnot_aux?

7. Is eta_practical_fraction declared as an assumption?

8. Is Q_aux_used limited by both available heat and conversion capacity?

9. Is P_aux_gross calculated from Q_aux_used and eta_aux_conversion?

10. Are power electronics losses included?

11. Are pump, fan, control, valve, and treatment loads included?

12. Is P_aux_net reported?

13. If P_aux_net is negative, is external / backup electricity required?

14. Is residual heat after conversion routed?

15. Does the model avoid claiming self-powered data center?

16. Does the model avoid claiming full SPHERE self-power without net-positive proof?

17. Are thermal kW and electric kW kept distinct?

18. Are 1 MW and 10 MW cases based on declared assumptions?

19. Is high-temperature conversion avoided unless high-temperature source is justified?

20. Which assumptions dominate feasibility: T_hot_aux, T_cold_aux, eta_practical_fraction, Q_aux_used, or internal electric load?
25. Minimum Site Data Required
available residual heat profile
temperature profile of residual heat
hot-side temperature range
cold-side / sink temperature range
hourly SPHERE heat allocation
W-CT electric load
A-DC electric load
W-env-DC electric load
thermal buffer electric load
heat export electric load
pump pressure drops
fluid flow rates
fan pressure drops
air flow rates
controls and sensors power demand
valves and actuators power demand
treatment equipment electric demand
critical control load
backup power architecture
desired auxiliary coverage target
available installation space
maintenance constraints
technology-specific conversion efficiency
power electronics efficiency
storage capacity
safety and electrical code constraints
26. Non-Claim Rules

Not allowed without site-specific proof:

self-powered data center
SPHERE fully powers itself
high electricity production from low-grade heat
zero external electricity for SPHERE
100% heat-to-electric conversion
Carnot efficiency achieved in practice
thermal kW counted as electric kW
residual heat disappears
backup power not required for critical controls
guaranteed economic payback

Allowed claim format:

Under declared assumptions:
SPHERE-DC routes X kW of heat to auxiliary conversion.
The hot side is A°C and cold side is B°C.
Carnot limit is C%.
Practical conversion efficiency is D%.
Gross auxiliary output is E kW.
Assigned internal load is F kW.
Net auxiliary balance is G kW.
Residual heat after conversion is H kW and must be routed.
27. Summary Formula Block
Q_aux_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export
- Q_buffer

Q_aux_available =
Q_captured × f_aux_heat

T_hot_aux_K =
T_hot_aux_C + 273.15

T_cold_aux_K =
T_cold_aux_C + 273.15

DeltaT_aux =
T_hot_aux_C - T_cold_aux_C

eta_Carnot_aux =
1 - (T_cold_aux_K / T_hot_aux_K)

eta_aux_conversion =
eta_Carnot_aux × eta_practical_fraction

Q_aux_used =
min(Q_aux_available, Q_aux_conversion_capacity)

P_aux_gross_raw =
Q_aux_used × eta_aux_conversion

P_aux_gross =
P_aux_gross_raw × eta_power_electronics

Q_aux_rejected =
Q_aux_used - P_aux_gross_raw

Q_power_electronics_heat =
P_aux_gross_raw - P_aux_gross

Q_aux_residual_total =
Q_aux_rejected + Q_power_electronics_heat

P_aux_internal_load =
P_pumps_total
+ P_fans_total
+ P_controls_total
+ P_valves_total
+ P_treatment_total
+ P_power_electronics_loss

P_aux_net =
P_aux_gross - P_aux_internal_load

E_aux_gross_day =
P_aux_gross × operating_hours_aux

E_aux_internal_load_day =
P_aux_internal_load × operating_hours_aux

E_aux_net_day =
E_aux_gross_day - E_aux_internal_load_day

P_pump_i =
(DeltaP_i × V_dot_i_m3_s) / eta_pump_i / 1000

P_pumps_total =
sum(P_pump_i)

P_fan_j =
(DeltaP_air_j × V_dot_air_j_m3_s) / eta_fan_j / 1000

P_fans_total =
sum(P_fan_j)

auxiliary_coverage_ratio =
P_aux_gross / P_aux_internal_load

SOC_aux_storage_next =
SOC_aux_storage
+ (P_to_storage × eta_storage_charge × Delta_t) / E_storage_aux
- (P_from_storage × Delta_t) / (E_storage_aux × eta_storage_discharge)