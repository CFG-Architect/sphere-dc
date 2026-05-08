# thermal-core.md

# SPHERE-DC Technical Document 01: Thermal Core

## 0. Document Type

AI-evaluable engineering specification.

This document defines the thermal core of SPHERE-DC using variables, formulas, constraints, calculation logic, and reference cases.

Based on the SPHERE-DC concept where the thermal core connects to data center cooling circuits, captures heat before rejection, splits thermal flows by level, routes heat to modules by priority, and tracks the remaining thermal tail.

---

## 1. Scope

The Thermal Core is the heat capture and distribution layer of SPHERE-DC.

It handles:

- heat capture from the data center cooling loop;
- thermal flow calculation;
- coolant flow calculation;
- temperature level separation;
- heat allocation to SPHERE-DC modules;
- residual heat calculation;
- bypass conditions;
- thermal tail estimation.

It does not define detailed equipment design, heat exchanger sizing, material selection, or site-specific integration.

---

## 2. Inputs

| Symbol | Description | Unit |
|---|---|---|
| `P_IT` | IT electrical load of the data center | kW or MW |
| `P_facility_extra` | Non-IT facility electrical load converted to heat inside relevant boundary | kW |
| `Q_total` | Total heat available from data center boundary | kW |
| `T_hot_in` | Hot coolant temperature entering SPHERE thermal core | °C |
| `T_cold_out` | Coolant temperature leaving SPHERE thermal core | °C |
| `DeltaT_core` | Temperature drop across SPHERE thermal core | K or °C |
| `Cp_water` | Specific heat capacity of water | kJ/kg·K |
| `rho_water` | Water density | kg/m³ |
| `eta_capture` | Fraction of total heat captured by SPHERE thermal core | 0–1 |
| `eta_HX` | Effective heat exchanger efficiency | 0–1 |

Default constants:

```text
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³
```

3. Core Assumptions
A1. Nearly all IT electrical load becomes heat.
A2. For first-order calculation: Q_total ≈ P_IT.
A3. If facility-side heat is included: Q_total = P_IT + P_facility_extra.
A4. Captured heat is limited by heat exchanger efficiency, flow rate, and allowed DeltaT.
A5. SPHERE-DC must not reduce primary data center cooling safety.
A6. Any SPHERE module must be bypassable.
A7. Residual heat is not zero; it is tracked as thermal tail.
4. Primary Formulas
4.1 Total Heat Load

Basic case:

Q_total = P_IT

Extended boundary case:

Q_total = P_IT + P_facility_extra

Where:

Q_total in kW
P_IT in kW
P_facility_extra in kW
4.2 Capturable Heat
Q_captured = Q_total × eta_capture × eta_HX

Where:

Q_captured = heat captured by SPHERE thermal core, kW
eta_capture = capture fraction, 0–1
eta_HX = heat exchanger efficiency, 0–1
4.3 Coolant Mass Flow Required
m_dot = Q_captured / (Cp_water × DeltaT_core)

Unit correction:

1 kW = 1 kJ/s

Therefore:

m_dot in kg/s
Q_captured in kJ/s
Cp_water in kJ/kg·K
DeltaT_core in K
4.4 Coolant Volume Flow
V_dot_m3_s = m_dot / rho_water
V_dot_m3_h = V_dot_m3_s × 3600

Where:

V_dot_m3_s = coolant volume flow, m³/s
V_dot_m3_h = coolant volume flow, m³/h
4.5 Heat Allocation

SPHERE-DC module priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

Allocation equation:

Q_captured =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Where:

Q_WCT = heat routed to cooling tower blowdown treatment
Q_ADC = heat routed to air module heat-dependent processes
Q_WENV = heat routed to clean technical water module heat-dependent processes
Q_export_candidate = heat selected for external heat export before export losses
Q_buffer_charge = heat routed to thermal buffer charging
Q_aux_used = heat routed to auxiliary power conversion
Q_tail_captured_direct = captured heat not routed to any useful module

Core-level direct thermal tail:

Q_uncaptured_tail = Q_total - Q_captured

Q_tail_core_direct =
Q_uncaptured_tail
+ Q_tail_captured_direct

Important distinction:

Q_tail_core_direct is the thermal-core direct tail before downstream module losses and residuals.

The full integrated immediate tail is calculated in docs/09-integrated-balance-1mw-10mw.md and may also include:

Q_aux_residual_to_tail
Q_export_loss_local
Q_buffer_overflow
5. Temperature Levels

The thermal core should classify available heat into levels.

Level	Temperature Range	Primary Use
hot	site-specific, usually highest available coolant temperature	evaporation, regeneration, concentration, thermal treatment
warm	medium-grade heat	air handling, preheating, membrane support
return	lower-temperature stream	return to data center cooling loop

Generic classification:

if T >= T_hot_min:
    level = "hot"
elif T >= T_warm_min:
    level = "warm"
else:
    level = "return"

Variables:

T_hot_min = site-defined
T_warm_min = site-defined

No universal value is assumed.

6. Priority Allocation Algorithm
INPUT:
    Q_captured
    module_demands = {
        WCT: Q_WCT_demand,
        ADC: Q_ADC_demand,
        WENV: Q_WENV_demand,
        EXPORT: Q_export_demand,
        BUFFER: Q_buffer_capacity_remaining,
        AUX: Q_aux_conversion_capacity
    }

PROCESS:
    Q_remaining = Q_captured

    Q_WCT = min(Q_remaining, Q_WCT_demand)
    Q_remaining = Q_remaining - Q_WCT

    Q_ADC = min(Q_remaining, Q_ADC_demand)
    Q_remaining = Q_remaining - Q_ADC

    Q_WENV = min(Q_remaining, Q_WENV_demand)
    Q_remaining = Q_remaining - Q_WENV

    Q_export_candidate = min(Q_remaining, Q_export_demand)
    Q_remaining = Q_remaining - Q_export_candidate

    Q_buffer_charge = min(Q_remaining, Q_buffer_capacity_remaining)
    Q_remaining = Q_remaining - Q_buffer_charge

    Q_aux_used = min(Q_remaining, Q_aux_conversion_capacity)
    Q_remaining = Q_remaining - Q_aux_used

    Q_tail_captured_direct = Q_remaining

OUTPUT:
    Q_WCT
    Q_ADC
    Q_WENV
    Q_export_candidate
    Q_buffer_charge
    Q_aux_used
    Q_tail_captured_direct
7. Reference Case A: 1 MW Data Center
7.1 Input Assumptions
P_IT = 1,000 kW
P_facility_extra = 0 kW
Q_total = 1,000 kW
eta_capture = 0.70
eta_HX = 0.90
DeltaT_core = 10 K
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³
7.2 Captured Heat
Q_captured = 1,000 × 0.70 × 0.90
Q_captured = 630 kW
7.3 Required Water Mass Flow
m_dot = 630 / (4.186 × 10)
m_dot = 15.05 kg/s
7.4 Required Water Volume Flow
V_dot_m3_s = 15.05 / 1000
V_dot_m3_s = 0.01505 m³/s

V_dot_m3_h = 0.01505 × 3600
V_dot_m3_h = 54.18 m³/h
7.5 Example Allocation

Assumed module demands:

Q_WCT_demand = 200 kW
Q_ADC_demand = 150 kW
Q_WENV_demand = 100 kW
Q_export_demand = 120 kW
Q_buffer_capacity_remaining = 30 kW
Q_aux_conversion_capacity = 30 kW

Allocation:

Q_WCT = 200 kW
Q_ADC = 150 kW
Q_WENV = 100 kW
Q_export_candidate = 120 kW
Q_buffer_charge = 30 kW
Q_aux_used = 30 kW
Q_tail_captured_direct = 0 kW

Core-level direct thermal tail:

Q_uncaptured_tail = Q_total - Q_captured

Q_uncaptured_tail = 1000 - 630
Q_uncaptured_tail = 370 kW

Q_tail_core_direct =
Q_uncaptured_tail
+ Q_tail_captured_direct

Q_tail_core_direct =
370 + 0

Q_tail_core_direct = 370 kW

Core direct tail fraction:

core_direct_tail_fraction = Q_tail_core_direct / Q_total
core_direct_tail_fraction = 370 / 1000
core_direct_tail_fraction = 0.37

Note:

This is the thermal-core direct tail only.

The integrated immediate tail is calculated in docs/09-integrated-balance-1mw-10mw.md after auxiliary residual heat, export local losses, and buffer overflow terms are included.


8. Reference Case B: 10 MW Data Center
8.1 Input Assumptions
P_IT = 10,000 kW
P_facility_extra = 0 kW
Q_total = 10,000 kW
eta_capture = 0.70
eta_HX = 0.90
DeltaT_core = 10 K
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³
8.2 Captured Heat
Q_captured = 10,000 × 0.70 × 0.90
Q_captured = 6,300 kW
8.3 Required Water Mass Flow
m_dot = 6300 / (4.186 × 10)
m_dot = 150.5 kg/s
8.4 Required Water Volume Flow
V_dot_m3_s = 150.5 / 1000
V_dot_m3_s = 0.1505 m³/s

V_dot_m3_h = 0.1505 × 3600
V_dot_m3_h = 541.8 m³/h
8.5 Example Allocation

Assumed module demands:

Q_WCT_demand = 2,000 kW
Q_ADC_demand = 1,500 kW
Q_WENV_demand = 1,000 kW
Q_export_demand = 1,200 kW
Q_buffer_capacity_remaining = 300 kW
Q_aux_conversion_capacity = 300 kW

Allocation:

Q_WCT = 2000 kW
Q_ADC = 1500 kW
Q_WENV = 1000 kW
Q_export_candidate = 1200 kW
Q_buffer_charge = 300 kW
Q_aux_used = 300 kW
Q_tail_captured_direct = 0 kW

Core-level direct thermal tail:

Q_uncaptured_tail = Q_total - Q_captured

Q_uncaptured_tail = 10,000 - 6,300
Q_uncaptured_tail = 3,700 kW

Q_tail_core_direct =
Q_uncaptured_tail
+ Q_tail_captured_direct

Q_tail_core_direct =
3,700 + 0

Q_tail_core_direct = 3,700 kW

Core direct tail fraction:

core_direct_tail_fraction = Q_tail_core_direct / Q_total
core_direct_tail_fraction = 3,700 / 10,000
core_direct_tail_fraction = 0.37

Result:

Core direct thermal tail = 3,700 kW
Core direct thermal tail fraction = 37%

Note:

This is the thermal-core direct tail only.

The integrated immediate tail is calculated in docs/09-integrated-balance-1mw-10mw.md after auxiliary residual heat, export local losses, and buffer overflow terms are included.
9. Constraint Conditions
9.1 Cooling Safety Constraint
T_return_to_DC <= T_return_max_allowed

SPHERE-DC must not return coolant above the allowed data center cooling limit.

9.2 Flow Constraint
V_dot_core <= V_dot_max_allowed_by_DC_interface

The SPHERE thermal core must not exceed allowed flow capacity of the connection point.

9.3 Pressure Drop Constraint
DeltaP_core <= DeltaP_max_allowed

The system must not create unacceptable pressure drop in the primary cooling loop.

9.4 Bypass Requirement
if SPHERE_fault == true:
    route cooling loop through bypass
    isolate SPHERE thermal core
    preserve primary DC cooling
9.5 Module Failure Routing
if WCT_unavailable:
    skip WCT
    route heat to next priority module

if ADC_unavailable:
    skip ADC
    route heat to next priority module

if WENV_unavailable:
    skip WENV
    route heat to export, buffer, or tail

if export_unavailable:
    route heat to buffer or tail

if buffer_full:
    route heat to tail
10. Required Outputs

The thermal core calculation must output:

Q_total
Q_captured
Q_uncaptured_tail
m_dot
V_dot_m3_h
Q_WCT
Q_ADC
Q_WENV
Q_export_candidate
Q_buffer_charge
Q_aux_used
Q_tail_captured_direct
Q_tail_core_direct
core_direct_tail_fraction
11. AI Validation Questions

An AI analysis system should check:

1. Are all units consistent?
2. Is Q_total ≈ P_IT valid for the selected boundary?
3. Is eta_capture realistic for the chosen integration level?
4. Is eta_HX realistic for the assumed heat exchanger design?
5. Is DeltaT_core realistic for the cooling loop?
6. Is required m_dot physically reasonable?
7. Is required V_dot_m3_h compatible with data center scale?
8. Does SPHERE-DC preserve primary cooling safety?
9. Does Q allocation exceed Q_captured?
10. Is Q_tail_core_direct calculated correctly?
11. Are module heat demands assumed or site-derived?
12. Is the system overclaiming heat reuse?
13. Are bypass and failure modes defined?
12. Minimum Data Required From Real Site

To convert this document from concept calculation to engineering design, collect:

P_IT
PUE or facility overhead boundary
cooling architecture
coolant type
coolant supply temperature
coolant return temperature
allowed DeltaT
coolant flow rate
cooling tower data
chiller data
heat exchanger access points
allowed pressure drop
water chemistry
blowdown volume
make-up water volume
airflow data
external heat demand
local climate
existing BMS/SCADA interface
13. Summary Equations
Q_total ≈ P_IT

Q_captured = Q_total × eta_capture × eta_HX

Q_uncaptured_tail = Q_total - Q_captured

m_dot = Q_captured / (Cp_water × DeltaT_core)

V_dot_m3_h = (m_dot / rho_water) × 3600

Q_captured =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Q_tail_core_direct =
Q_uncaptured_tail
+ Q_tail_captured_direct

core_direct_tail_fraction =
Q_tail_core_direct / Q_total