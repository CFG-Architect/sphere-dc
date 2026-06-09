# assumptions-and-variables.md

# SPHERE-DC Technical Document 02: Assumptions and Variables

## 0. Document Type

AI-evaluable engineering specification.

This document defines shared assumptions, variables, units, constants, and base formulas used across all SPHERE-DC technical documents.

Purpose:

```text
avoid duplicated definitions
keep module calculations consistent
make AI validation easier
make formulas machine-readable
separate assumptions from claims
```
## Scope Warning

This document defines shared variables, formulas, and assumptions only.

It does not contain:
- module-level calculation logic
- module-level reference cases  
- cross-module consistency checks
- failure and bypass conditions
- site-specific constraints per module

Reading this document does not substitute for reading docs 03–09.
Each module document contains specifications, constraints, and failure conditions
not present here.

1. System Boundary

SPHERE-DC is modeled as an engineering layer connected to a data center.

Primary data center functions remain outside SPHERE-DC.

SPHERE-DC interacts with:

thermal flows
cooling water flows
cooling tower blowdown
air flows
condensate
technical water
heat export interfaces
auxiliary energy recovery
thermal buffers

SPHERE-DC must not compromise primary data center cooling safety.

2. Reference Scales

Default reference cases:

Case A: 1 MW IT load
Case B: 10 MW IT load

These are calculation cases, not deployment limits.

3. Global Assumptions
A1. Nearly all IT electrical load becomes heat.

A2. First-order thermal load:
    Q_total ≈ P_IT

A3. Extended thermal boundary:
    Q_total = P_IT + P_facility_extra

A4. SPHERE-DC captures only a fraction of total available heat.

A5. All heat reuse claims must be expressed as allocated heat flows.

A6. Residual heat is never zero.
    Remaining unused heat is tracked as Q_tail_total.

A7. Water recovery is never 100%.
    Remaining brine, sludge, reject, or waste stream must be tracked.

A8. Auxiliary power generation is secondary.
    It supports SPHERE-DC internal loads and must not be modeled as powering the entire data center.

A9. Each module must be bypassable.

A10. Data center cooling safety has priority over SPHERE-DC process efficiency.

A11. All numerical assumptions must be explicitly declared.

A12. Site-specific values must replace defaults before engineering design.
4. Unit Standard

Use SI units.

Preferred units:

Power: kW, MW
Energy: kWh, MWh, MJ
Temperature: °C, K
Temperature difference: K
Mass flow: kg/s
Volume flow: m³/s, m³/h, m³/day
Water quality: mg/L, ppm, µS/cm
Pressure: Pa, kPa, bar
Air flow: m³/h
Humidity ratio: kg_water/kg_dry_air
Efficiency: 0–1
Percentage: %
Time: s, h, day

Conversion rules:

1 MW = 1000 kW
1 kW = 1 kJ/s
1 m³ water ≈ 1000 kg
1 day = 24 h = 86400 s
5. Default Physical Constants
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³

Cp_air = 1.005 kJ/kg·K
rho_air = 1.2 kg/m³

h_fg_water_evap_default = 2450 kJ/kg

Notes:

Cp_water may vary with temperature and chemistry.
rho_water may vary with salinity and temperature.
rho_air depends on temperature, pressure, and humidity.
h_fg_water_evap_default is a first-order estimate for cooling tower calculations.
6. Global Thermal Variables
Symbol	Meaning	Unit
P_IT	IT electrical load	kW
P_facility_extra	Additional facility load converted to heat inside boundary	kW
Q_total	Total heat load inside selected boundary	kW
Q_captured	Heat captured by SPHERE-DC thermal core	kW
Q_used_total	Total heat routed to useful modules	kW
Q_tail_total	Total residual unused heat	kW
eta_capture	Heat capture fraction	0–1
eta_HX	Effective heat exchanger efficiency	0–1
DeltaT_core	Temperature drop across thermal core	K
T_hot_in	Hot coolant temperature entering SPHERE-DC	°C
T_cold_out	Coolant temperature leaving SPHERE-DC	°C
m_dot_core	Coolant mass flow through thermal core	kg/s
V_dot_core	Coolant volume flow through thermal core	m³/h

Base formulas:

Q_total ≈ P_IT

Q_total = P_IT + P_facility_extra

Q_captured = Q_total × eta_capture × eta_HX

m_dot_core = Q_captured / (Cp_water × DeltaT_core)

V_dot_core_m3_s = m_dot_core / rho_water

V_dot_core_m3_h = V_dot_core_m3_s × 3600
7. Heat Allocation Variables

SPHERE-DC heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail
Symbol	Meaning	Unit
Q_WCT	Heat routed to cooling tower blowdown module	kW
Q_ADC	Heat routed to air handling module	kW
Q_WENV	Heat routed to clean technical water module	kW
Q_export	Heat routed to external heat user	kW
Q_buffer	Heat stored in buffer	kW
Q_tail_captured	Captured heat not used by modules	kW

Balance:

Q_captured = Q_WCT + Q_ADC + Q_WENV + Q_export + Q_buffer + Q_tail_captured

Q_used_total = Q_WCT + Q_ADC + Q_WENV + Q_export + Q_buffer

Q_tail_total = Q_total - Q_used_total

tail_fraction = Q_tail_total / Q_total

Constraint:

Q_used_total <= Q_total
Q_WCT + Q_ADC + Q_WENV + Q_export + Q_buffer <= Q_captured
8. Cooling Tower / W-CT Variables
Symbol	Meaning	Unit
Q_CT	Heat load rejected through cooling tower boundary	kW
V_circ_CT	Cooling tower circulating water flow	m³/day
V_evap	Evaporative water loss	m³/day
V_blowdown	Cooling tower blowdown volume	m³/day
V_makeup	Required make-up water	m³/day
V_drift	Drift loss	m³/day
V_leak	Leakage or miscellaneous loss	m³/day
CoC	Cycles of concentration	dimensionless
R_WCT	Water recovery rate from blowdown	0–1
V_WCT_recovered	Recovered technical water from blowdown	m³/day
V_WCT_waste	Remaining brine/sludge/reject volume	m³/day
TDS_in	Total dissolved solids in blowdown	mg/L
TDS_out	TDS in recovered water	mg/L
Q_WCT_required	Heat demand of W-CT module	kW
P_WCT_electric	Electric load of W-CT module	kW

First-order evaporation estimate:

m_evap_kg_s = Q_CT / h_fg_water_evap_default

V_evap_m3_day = (m_evap_kg_s / rho_water) × 86400

Simplified blowdown estimate:

V_blowdown = V_evap / (CoC - 1)

Extended make-up water balance:

V_makeup = V_evap + V_blowdown + V_drift + V_leak

Recovery:

V_WCT_recovered = V_blowdown × R_WCT

V_WCT_waste = V_blowdown - V_WCT_recovered

Constraints:

CoC > 1
0 <= R_WCT <= 1
V_WCT_recovered <= V_blowdown
V_WCT_waste >= 0
9. Air / A-DC Variables
Symbol	Meaning	Unit
V_dot_air	Air volume flow	m³/h
m_dot_air	Dry air mass flow	kg/s
rho_air	Air density	kg/m³
T_air_in	Air temperature at module inlet	°C
T_air_out	Air temperature at module outlet	°C
RH_in	Relative humidity at inlet	%
RH_out	Relative humidity at outlet	%
omega_in	Inlet humidity ratio	kg_water/kg_dry_air
omega_out	Outlet humidity ratio	kg_water/kg_dry_air
m_dot_condensate	Condensate mass flow	kg/s
V_condensate	Condensate volume	m³/day
DeltaP_filter	Filter pressure drop	Pa
eta_fan	Fan efficiency	0–1
P_fan	Fan electric power	kW
PM_in	Particle concentration at inlet	µg/m³
PM_out	Particle concentration at outlet	µg/m³
eta_filter_PM	Particle removal efficiency	0–1

Air mass flow:

m_dot_air = (rho_air × V_dot_air) / 3600

Condensate:

m_dot_condensate = m_dot_air × max(0, omega_in - omega_out)

V_condensate_m3_day = (m_dot_condensate / rho_water) × 86400

Fan power:

P_fan_kW = (DeltaP_filter × V_dot_air_m3_s) / eta_fan / 1000

Particle removal:

eta_filter_PM = (PM_in - PM_out) / PM_in

Constraints:

0 <= RH_in <= 100
0 <= RH_out <= 100
omega_out <= omega_in only if dehumidification occurs
PM_out <= PM_in
0 < eta_fan <= 1
10. Clean Technical Water / W-env-DC Variables
Symbol	Meaning	Unit
V_WCT_recovered	Water recovered from W-CT	m³/day
V_condensate	Condensate recovered from A-DC	m³/day
V_external	External water input	m³/day
V_WENV_in	Total water entering W-env-DC	m³/day
R_WENV	Recovery rate of clean technical water module	0–1
V_clean_technical	Clean technical water produced	m³/day
V_WENV_reject	Reject stream from W-env-DC	m³/day
Q_WENV_required	Heat demand of W-env-DC	kW
P_WENV_electric	Electric load of W-env-DC	kW

Input balance:

V_WENV_in = V_WCT_recovered + V_condensate + V_external

Recovery:

V_clean_technical = V_WENV_in × R_WENV

V_WENV_reject = V_WENV_in - V_clean_technical

Constraints:

0 <= R_WENV <= 1
V_clean_technical <= V_WENV_in
V_WENV_reject >= 0
11. Thermal Buffer Variables
Symbol	Meaning	Unit
Q_peak_excess	Excess heat during peak	kW
t_peak	Peak duration	h
E_buffer_required	Required buffer energy capacity	kWh
E_buffer_available	Available buffer energy capacity	kWh
eta_buffer_charge	Buffer charge efficiency	0–1
eta_buffer_discharge	Buffer discharge efficiency	0–1
m_buffer_water	Water mass used as buffer	kg
DeltaT_buffer	Buffer temperature swing	K
m_PCM	Phase-change material mass	kg
L_PCM	Latent heat of PCM	kJ/kg

Energy capacity:

E_buffer_required = Q_peak_excess × t_peak

Water buffer:

E_water_kJ = m_buffer_water × Cp_water × DeltaT_buffer

E_water_kWh = E_water_kJ / 3600

PCM buffer:

E_PCM_kJ = m_PCM × L_PCM

E_PCM_kWh = E_PCM_kJ / 3600

Usable buffer output:

E_buffer_usable = E_buffer_available × eta_buffer_charge × eta_buffer_discharge

Constraints:

E_buffer_usable >= required peak coverage target
0 < eta_buffer_charge <= 1
0 < eta_buffer_discharge <= 1
12. Heat Export Variables
Symbol	Meaning	Unit
Q_export_available	Heat available for export	kW
Q_export_demand	External heat demand	kW
Q_export_delivered	Heat delivered to external user	kW
Q_export_losses	Heat lost during transfer	kW
T_export_supply	Export supply temperature	°C
T_export_return	Export return temperature	°C
distance_export	Distance to heat user	m
eta_export	Export delivery efficiency	0–1

Delivered heat:

Q_export_delivered = min(Q_export_available, Q_export_demand) × eta_export

Losses:

Q_export_losses = min(Q_export_available, Q_export_demand) - Q_export_delivered

Validity condition:

Heat export is useful only if:
Q_export_delivered replaces existing heat demand.

Constraint:

0 <= eta_export <= 1
Q_export_delivered <= Q_export_available
Q_export_delivered <= Q_export_demand
13. Auxiliary Power Variables
Symbol	Meaning	Unit
Q_aux_used	Heat routed to auxiliary power generation	kW
eta_aux_conversion	Heat-to-electric conversion efficiency	0–1
P_aux_gross	Gross auxiliary electric output	kW
P_pumps	Pump electric load	kW
P_fans	Fan electric load	kW
P_controls	Sensors, control, automation load	kW
P_aux_internal_load	Total SPHERE internal electric load	kW
P_aux_net	Net auxiliary electric balance	kW

Gross auxiliary power:

P_aux_gross = Q_aux_used × eta_aux_conversion

Internal electric load:

P_aux_internal_load = P_pumps + P_fans + P_controls

Net auxiliary balance:

P_aux_net = P_aux_gross - P_aux_internal_load

Constraints:

0 <= eta_aux_conversion <= 1
P_aux_gross <= Q_aux_used
Auxiliary power must not be modeled as primary data center power.
14. System-Level Output Variables
Symbol	Meaning	Unit
water_reuse_total	Total recovered water reused	m³/day
fresh_water_reduction	Reduction of external fresh water demand	m³/day
liquid_discharge_reduction	Reduction of liquid discharge	m³/day
thermal_tail_reduction	Reduction of unused local heat output	kW
aux_power_net	Net auxiliary electricity	kW
waste_concentrate_total	Total brine/sludge/reject output	m³/day
system_electric_load	Total SPHERE electric load	kW

Water reuse:

water_reuse_total = V_WCT_recovered + V_condensate + V_clean_technical

Fresh water reduction:

fresh_water_reduction = min(water_reuse_total, original_makeup_water_demand)

Liquid discharge reduction:

liquid_discharge_reduction = original_liquid_discharge - new_liquid_discharge

Thermal tail reduction:

thermal_tail_reduction = original_thermal_tail - Q_tail_total

Waste concentrate:

waste_concentrate_total = V_WCT_waste + V_WENV_reject
15. Default Reference Assumption Set

Use only for early concept calculation.

P_IT_case_A = 1,000 kW
P_IT_case_B = 10,000 kW

P_facility_extra = 0 kW

eta_capture = 0.70
eta_HX = 0.90
DeltaT_core = 10 K

Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³

Cp_air = 1.005 kJ/kg·K
rho_air = 1.2 kg/m³

h_fg_water_evap_default = 2450 kJ/kg

CoC_default = site-specific
R_WCT_default = site-specific
R_WENV_default = site-specific

eta_aux_conversion = technology-specific
eta_export = site-specific

Values marked site-specific or technology-specific must not be assumed in final engineering claims.

16. Required Site Data

Before engineering design, collect:

P_IT
PUE
cooling system type
coolant type
coolant supply temperature
coolant return temperature
coolant flow rate
allowed DeltaT
allowed pressure drop
heat exchanger access points

cooling tower heat rejection
cooling tower circulation rate
cooling tower make-up water
cooling tower blowdown
cycles of concentration
drift loss
water chemistry
TDS
pH
biocides
corrosion inhibitors
metals

airflow rates
air temperatures
relative humidity
particle concentration
filter data
fan pressure drop

external water cost
wastewater discharge cost
available heat users
heat export distance
local climate
electricity price
environmental constraints
BMS/SCADA integration data
17. AI Validation Checklist

An AI analysis system should verify:

1. Are all variables defined before use?

2. Are all units consistent?

3. Does Q_total match the selected system boundary?

4. Is Q_total ≈ P_IT acceptable for first-order analysis?

5. Are eta_capture and eta_HX declared as assumptions, not facts?

6. Does any module use more heat than Q_captured?

7. Is Q_tail_total always non-negative?

8. Is water recovery less than or equal to input water?

9. Is waste/reject/sludge tracked?

10. Is blowdown calculated with valid CoC > 1?

11. Is auxiliary power modeled as support power, not primary data center power?

12. Are fan and pump loads subtracted from auxiliary generation?

13. Is heat export counted only when there is real external demand?

14. Are site-specific values clearly marked?

15. Are default values prevented from becoming engineering claims?

16. Is primary data center cooling safety preserved?

17. Are bypass conditions required for every module?

18. Does the model avoid claiming zero heat or zero discharge?

19. Are 1 MW and 10 MW reference cases scalable from the same formulas?

20. Which assumptions dominate the result sensitivity?
18. Non-Claim Rules

The following claims are not allowed unless supported by site-specific calculations:

zero heat
zero discharge
fully green data center
self-powered data center
100% water recovery
complete elimination of cooling tower waste
universal economic viability
guaranteed payback

Allowed claim format:

Under declared assumptions, the model estimates:
- captured heat
- reused heat
- recovered water
- remaining thermal tail
- remaining waste stream
- auxiliary power balance
19. Summary Formula Block
Q_total ≈ P_IT

Q_total = P_IT + P_facility_extra

Q_captured = Q_total × eta_capture × eta_HX

m_dot_core = Q_captured / (Cp_water × DeltaT_core)

V_dot_core_m3_h = (m_dot_core / rho_water) × 3600

Q_captured = Q_WCT + Q_ADC + Q_WENV + Q_export + Q_buffer + Q_tail_captured

Q_tail_total = Q_total - Q_WCT - Q_ADC - Q_WENV - Q_export - Q_buffer

tail_fraction = Q_tail_total / Q_total

m_evap_kg_s = Q_CT / h_fg_water_evap_default

V_evap_m3_day = (m_evap_kg_s / rho_water) × 86400

V_blowdown = V_evap / (CoC - 1)

V_WCT_recovered = V_blowdown × R_WCT

V_WCT_waste = V_blowdown - V_WCT_recovered

m_dot_air = (rho_air × V_dot_air) / 3600

m_dot_condensate = m_dot_air × max(0, omega_in - omega_out)

V_condensate_m3_day = (m_dot_condensate / rho_water) × 86400

V_WENV_in = V_WCT_recovered + V_condensate + V_external

V_clean_technical = V_WENV_in × R_WENV

V_WENV_reject = V_WENV_in - V_clean_technical

E_buffer_required = Q_peak_excess × t_peak

Q_export_delivered = min(Q_export_available, Q_export_demand) × eta_export

P_aux_gross = Q_aux_used × eta_aux_conversion

P_aux_net = P_aux_gross - P_pumps - P_fans - P_controls
