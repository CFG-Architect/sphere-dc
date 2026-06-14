# integrated-balance-1mw-10mw.md

# SPHERE-DC Technical Document 09: Integrated Balance for 1 MW and 10 MW Reference Cases

## 0. Document Type

AI-evaluable integrated engineering specification.

System:

```text
SPHERE-DC = thermal-water-air-energy integration layer around a data center

Reference scales:

Case A: 1 MW IT data center
Case B: 10 MW IT data center

Purpose:

This document connects the previous SPHERE-DC modules into one integrated system balance.

It checks:
- heat capture
- heat allocation
- thermal tail
- water recovery
- clean technical water production
- liquid reject
- heat export
- thermal buffer contribution
- auxiliary power
- module interaction consistency
- double-counting risk
```

1. Included Technical Documents

This integrated balance uses variables and formulas from:

01 assumptions-and-variables.md
02 thermal-core.md
03 w-ct.md
04 a-dc.md
05 w-env-dc.md
06 thermal-buffers.md
07 heat-export.md
08 auxiliary-power.md
2. System Boundary
2.1 Included
IT electrical load converted to heat
captured data center heat
uncaptured thermal tail
cooling tower blowdown recovery
A-DC condensate capture
W-env-DC technical water production
heat export
thermal buffering
limited auxiliary heat-to-electric conversion
SPHERE internal electric support loads
final water reject streams
2.2 Excluded Unless Site-Specified
full chiller plant design
complete HVAC system design
complete district heating network
potable water certification
final waste classification
civil engineering
commercial payback
legal permitting
full life-cycle analysis
3. Core Non-Claim Rules
SPHERE-DC does not eliminate heat.

SPHERE-DC does not create zero thermal tail.

SPHERE-DC does not create zero liquid discharge by default.

SPHERE-DC does not produce potable water by default.

SPHERE-DC does not self-power the data center.

SPHERE-DC auxiliary power is support power only.

Recovered water is not clean technical water until treated or validated.

Exported heat is useful only if it replaces real external heat demand.

Stored heat is not eliminated; it is delayed until later use or rejection.
4. Integrated Heat Balance Structure
4.1 Total Heat

First-order:

Q_total ≈ P_IT

Extended:

Q_total = P_IT + P_facility_extra

Reference cases use:

P_facility_extra = 0
Q_total = P_IT
4.2 Captured Heat
Q_captured = Q_total × eta_capture × eta_HX

Uncaptured heat:

Q_uncaptured_tail = Q_total - Q_captured
4.3 Captured Heat Allocation

Integrated captured heat allocation:

Q_captured =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Constraint:

Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct
<= Q_captured

If the sum is lower than Q_captured:

Q_tail_captured_direct =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export_candidate
- Q_buffer_charge
- Q_aux_used

If the sum is higher than Q_captured:

integrated_balance_status = "invalid_overallocated_heat"
5. Immediate Thermal Tail

Immediate local thermal tail:

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

Where:

Q_aux_residual_to_tail <= Q_aux_residual_total
Q_export_loss_local <= Q_loss_HX
Q_buffer_overflow >= 0

Interpretation:

Q_tail_immediate is not the same as total heat conservation.
It estimates heat still requiring local rejection after useful routing and temporary storage.
6. Full-Cycle Thermal Accounting

Full-cycle rule:

All heat eventually exits as:
- useful external heat
- process heat
- stored heat later discharged
- local thermal tail
- external distribution loss
- residual conversion heat
- emitted or rejected heat elsewhere

Therefore:

Do not report thermal buffer charge, heat export, W-CT heat use, A-DC heat use, or W-env-DC heat use as heat destruction.

Allowed metric:

immediate_local_tail_reduction

Not allowed without full time-series proof:

permanent_heat_elimination
7. Integrated Water Balance Structure
7.1 Cooling Tower Baseline
V_makeup_original =
V_evap
+ V_blowdown_original
+ V_drift
+ V_leak

If drift and leak are not modeled:

V_makeup_original ≈ V_evap + V_blowdown_original
7.2 W-CT Recovery
V_WCT_recovered =
V_blowdown_original × R_WCT
V_WCT_waste =
V_blowdown_original - V_WCT_recovered
7.3 A-DC Condensate
V_condensate_captured =
V_condensate × R_condensate_capture
7.4 W-env-DC Input
V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water
7.5 W-env-DC Output
V_WENV_treated_clean =
V_WENV_treatment_in × R_WENV
V_WENV_reject =
V_WENV_treatment_in - V_WENV_treated_clean
V_clean_technical =
V_WENV_treated_clean + V_WENV_bypass_clean
7.6 Final Liquid Reject
V_final_liquid_reject =
V_WCT_waste
+ V_WENV_reject
+ V_other_reject

If no other reject is modeled:

V_final_liquid_reject =
V_WCT_waste
+ V_WENV_reject
7.7 Fresh Water Reduction
fresh_water_reduction =
min(V_clean_technical, V_makeup_original)
V_external_makeup_new =
V_makeup_original - fresh_water_reduction

Constraint:

V_external_makeup_new >= 0
8. Integrated Electric Balance Structure
8.1 Known Module Electric Loads
P_known_module_loads =
P_WCT_electric
+ P_ADC_electric
+ P_WENV_electric
+ P_export_electric
+ P_buffer_electric
+ P_controls_total
+ P_other_SPHERE_loads

If some module loads are unknown:

P_known_module_loads is a partial subtotal.
Unknown loads must remain explicit placeholders.
8.2 Auxiliary Power
P_aux_gross =
Q_aux_used × eta_aux_conversion × eta_power_electronics

P_aux_net =
P_aux_gross - P_aux_internal_load

Integrated external electricity requirement:

P_external_SPHERE_required =
max(0, P_known_module_loads - P_aux_gross)

Interpretation:

P_known_module_loads is the total known SPHERE-DC electric demand inside the selected boundary.

P_aux_internal_load is not an additional demand added on top of P_known_module_loads.

P_aux_internal_load is the subset of P_known_module_loads that auxiliary power attempts to offset.

Therefore:

P_external_SPHERE_required must not be calculated as:

P_known_module_loads + P_aux_internal_load - P_aux_gross

Correct integrated logic:

P_external_SPHERE_required =
max(0, P_known_module_loads - P_aux_gross)

Double-counting rule:

A pump, fan, control system, valve, treatment unit, power electronics loss, or other electric load must appear in exactly one demand subtotal.

The same electric load may be assigned to auxiliary offset accounting, but it must not be added twice as a separate demand.
9. Reference Case A: 1 MW Integrated Balance
9.1 Declared Global Inputs
P_IT = 1,000 kW
P_facility_extra = 0 kW

Q_total = 1,000 kW

eta_capture = 0.70
eta_HX = 0.90

Q_captured = 630 kW
Q_uncaptured_tail = 370 kW
9.2 Declared Heat Allocation
Q_WCT = 200 kW
Q_ADC = 150 kW
Q_WENV = 100 kW
Q_export_candidate = 120 kW
Q_buffer_charge = 30 kW
Q_aux_used = 30 kW
Q_tail_captured_direct = 0 kW

Allocation check:

Q_allocated =
200 + 150 + 100 + 120 + 30 + 30 + 0

Q_allocated = 630 kW
Q_allocated = Q_captured

heat_allocation_status = "valid"

A-DC interpretation note:

In this integrated balance, Q_ADC means heat allocated from the captured SPHERE thermal core to A-DC heat-dependent processes.

Q_ADC does not represent the full A-DC air-side thermal load.

The full A-DC air-side thermal interaction is represented in the A-DC module as Q_ADC_total.

Therefore, Q_ADC_total may exceed Q_ADC without invalidating the integrated heat allocation, provided that the excess air-side load is handled by existing HVAC systems, auxiliary energy, reduced A-DC process load, reduced airflow, changed humidity targets, or bypass.

W-env-DC interpretation note:

In this integrated balance, Q_WENV means heat allocated from the captured SPHERE thermal core to W-env-DC heat-dependent processes.

Q_WENV does not prove that the full declared clean technical water output is produced by pure thermal evaporation or distillation.

The W-env-DC module defines multiple possible thermal demand modes, including thermal evaporation / distillation and low-thermal / membrane-support treatment.

Pure thermal evaporation may require significantly more heat than the declared Q_WENV allocation.

Therefore, the integrated balance assumes only declared heat allocation to W-env-DC, not final process technology selection.

If actual Q_WENV_required exceeds Q_WENV, the unresolved demand must be handled by reduced throughput, reduced recovery rate, heat recovery, auxiliary heat, lower-thermal treatment, external treatment, storage, or bypass.
9.3 Heat Export Sub-Balance

Declared from heat export reference case:

Q_export_candidate = 120 kW
eta_HX_export = 0.95
Q_after_HX = 114 kW
Q_loss_HX = 6 kW
Q_loss_pipe = 0.55 kW
Q_export_delivered = 113.45 kW
Q_export_demand = 120 kW
Q_export_useful = 113.45 kW
Q_export_rejected = 0 kW

Daily useful export:

operating_hours_export = 12 h/day

E_export_useful_day =
113.45 × 12

E_export_useful_day = 1361.4 kWh_th/day
9.4 Thermal Buffer Sub-Balance

Declared:

Q_buffer_charge = 30 kW

From thermal buffer sizing example:

Q_peak_excess = 150 kW
t_peak = 2 h
safety_factor_buffer = 1.20

E_buffer_required = 360 kWh usable
V_buffer_water ≈ 15.5 m³ at DeltaT_buffer = 20 K

Interpretation:

The integrated heat allocation charges the buffer at 30 kW.
The peak-sizing example shows required capacity for a separate 150 kW, 2-hour peak.
Do not confuse buffer charge power with total buffer capacity.
9.5 Auxiliary Power Sub-Balance

Declared from auxiliary-power reference case:

Q_aux_used = 30 kW

T_hot_aux_C = 80°C
T_cold_aux_C = 30°C

eta_Carnot_aux = 0.1416
eta_practical_fraction = 0.15
eta_aux_conversion = 0.0212
eta_power_electronics = 0.95

P_aux_gross_raw = 0.636 kW
P_aux_gross = 0.604 kW

P_aux_internal_load = 7.5 kW
P_aux_net = -6.896 kW

Q_aux_residual_total = 29.396 kW

Residual routing assumption:

Q_aux_residual_to_tail = 29.396 kW
9.6 Immediate Thermal Tail

Assume:

Q_export_loss_local = Q_loss_HX = 6 kW
Q_buffer_overflow = 0 kW
Q_tail_captured_direct = 0 kW
Q_aux_residual_to_tail = 29.396 kW

Then:

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

Q_tail_immediate =
370 + 0 + 29.396 + 6 + 0

Q_tail_immediate = 405.396 kW

Thermal interpretation:

Immediate local tail is approximately 405 kW under declared assumptions.
This is not permanent heat elimination.
It excludes heat exported usefully, heat temporarily buffered, and process heat treated as assigned module use.
9.7 W-CT Water Sub-Balance

Declared from W-CT reference case:

V_evap = 35.3 m³/day
V_blowdown_original = 17.65 m³/day
V_makeup_original = 52.95 m³/day

R_WCT = 0.75

V_WCT_recovered = 13.24 m³/day
V_WCT_waste = 4.41 m³/day
9.8 A-DC Condensate Sub-Balance

Declared from A-DC reference case:

V_condensate = 25.2 m³/day
R_condensate_capture = 0.90
V_condensate_captured = 22.7 m³/day
V_condensate_lost = 2.5 m³/day
9.9 W-env-DC Water Sub-Balance

Declared:

V_WCT_recovered = 13.24 m³/day
V_condensate_captured = 22.7 m³/day
V_external = 0 m³/day
V_rainwater = 0 m³/day
V_other_water = 0 m³/day

R_WENV = 0.90
V_WENV_bypass_clean = 0 m³/day

Input:

V_WENV_in =
13.24 + 22.7

V_WENV_in = 35.94 m³/day

Clean technical water:

V_clean_technical =
35.94 × 0.90

V_clean_technical = 32.35 m³/day

Reject:

V_WENV_reject =
35.94 - 32.35

V_WENV_reject = 3.59 m³/day
9.10 Final Water Balance

Final liquid reject:

V_final_liquid_reject =
V_WCT_waste + V_WENV_reject

V_final_liquid_reject =
4.41 + 3.59

V_final_liquid_reject = 8.00 m³/day

Fresh water reduction:

fresh_water_reduction =
min(32.35, 52.95)

fresh_water_reduction = 32.35 m³/day

New external make-up requirement:

V_external_makeup_new =
52.95 - 32.35

V_external_makeup_new = 20.60 m³/day

Make-up replacement fraction:

makeup_replacement_fraction =
32.35 / 52.95

makeup_replacement_fraction = 0.611

Liquid reject reduction relative to original blowdown:

liquid_reject_reduction =
V_blowdown_original - V_final_liquid_reject

liquid_reject_reduction =
17.65 - 8.00

liquid_reject_reduction = 9.65 m³/day
liquid_reject_reduction_fraction =
9.65 / 17.65

liquid_reject_reduction_fraction = 0.547

Boundary note:

V_final_liquid_reject includes W-env-DC reject generated from both W-CT recovered water and A-DC condensate.
Therefore, liquid reject reduction must be reported with boundary definition.
9.11 Electric Balance

Known declared loads:

P_ADC_fan = 21.4 kW
P_export_electric = 0.676 kW
P_aux_internal_load = 7.5 kW

Reference-case interpretation:

In this reference subtotal, P_aux_internal_load represents additional known SPHERE-DC support loads not already counted as P_ADC_fan or P_export_electric.

It must not include the same A-DC fan load or export pump/control load already listed above.
Known subtotal:

P_known_module_loads =
21.4 + 0.676 + 7.5

P_known_module_loads = 29.576 kW

Auxiliary gross:

P_aux_gross = 0.604 kW

Known external electricity required:

P_external_SPHERE_required_known =
max(0, 29.576 - 0.604)

P_external_SPHERE_required_known = 28.972 kW

Daily known electricity:

E_external_SPHERE_required_known_day =
28.972 × 24

E_external_SPHERE_required_known_day = 695.3 kWh/day

Missing loads:

P_WCT_electric = site-specific
P_WENV_electric = site-specific
P_buffer_electric = site-specific
P_other_SPHERE_loads = site-specific

Interpretation:

The 1 MW integrated reference is not electrically self-sufficient.
Auxiliary power offsets only a small fraction of known support loads.
10. Reference Case B: 10 MW Integrated Balance
10.1 Declared Global Inputs
P_IT = 10,000 kW
P_facility_extra = 0 kW

Q_total = 10,000 kW

eta_capture = 0.70
eta_HX = 0.90

Q_captured = 6,300 kW
Q_uncaptured_tail = 3,700 kW
10.2 Declared Heat Allocation
Q_WCT = 2,000 kW
Q_ADC = 1,500 kW
Q_WENV = 1,000 kW
Q_export_candidate = 1,200 kW
Q_buffer_charge = 300 kW
Q_aux_used = 300 kW
Q_tail_captured_direct = 0 kW

Allocation check:

Q_allocated =
2000 + 1500 + 1000 + 1200 + 300 + 300 + 0

Q_allocated = 6300 kW
Q_allocated = Q_captured

heat_allocation_status = "valid"
10.3 Heat Export Sub-Balance

Declared from heat export reference case:

Q_export_candidate = 1,200 kW
eta_HX_export = 0.95
Q_after_HX = 1,140 kW
Q_loss_HX = 60 kW
Q_loss_pipe = 3.53 kW
Q_export_delivered = 1,136.47 kW
Q_export_demand = 1,200 kW
Q_export_useful = 1,136.47 kW
Q_export_rejected = 0 kW

Daily useful export:

operating_hours_export = 12 h/day

E_export_useful_day =
1136.47 × 12

E_export_useful_day = 13,637.6 kWh_th/day
10.4 Thermal Buffer Sub-Balance

Declared:

Q_buffer_charge = 300 kW

From thermal buffer sizing example:

Q_peak_excess = 1,500 kW
t_peak = 2 h
safety_factor_buffer = 1.20

E_buffer_required = 3,600 kWh usable
V_buffer_water ≈ 155 m³ at DeltaT_buffer = 20 K

Interpretation:

The integrated heat allocation charges the buffer at 300 kW.
The peak-sizing example shows required capacity for a separate 1,500 kW, 2-hour peak.
10.5 Auxiliary Power Sub-Balance

Declared from auxiliary-power reference case:

Q_aux_used = 300 kW

T_hot_aux_C = 80°C
T_cold_aux_C = 30°C

eta_Carnot_aux = 0.1416
eta_practical_fraction = 0.15
eta_aux_conversion = 0.0212
eta_power_electronics = 0.95

P_aux_gross_raw = 6.36 kW
P_aux_gross = 6.04 kW

P_aux_internal_load = 50 kW
P_aux_net = -43.96 kW

Q_aux_residual_total = 293.96 kW

Residual routing assumption:

Q_aux_residual_to_tail = 293.96 kW
10.6 Immediate Thermal Tail

Assume:

Q_export_loss_local = Q_loss_HX = 60 kW
Q_buffer_overflow = 0 kW
Q_tail_captured_direct = 0 kW
Q_aux_residual_to_tail = 293.96 kW

Then:

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

Q_tail_immediate =
3700 + 0 + 293.96 + 60 + 0

Q_tail_immediate = 4053.96 kW

Thermal interpretation:

Immediate local tail is approximately 4.05 MW under declared assumptions.
This is not permanent heat elimination.
It excludes heat exported usefully, heat temporarily buffered, and process heat treated as assigned module use.
10.7 W-CT Water Sub-Balance

Declared from W-CT reference case:

V_evap = 352.8 m³/day
V_blowdown_original = 176.4 m³/day
V_makeup_original = 529.2 m³/day

R_WCT = 0.75

V_WCT_recovered = 132.3 m³/day
V_WCT_waste = 44.1 m³/day
10.8 A-DC Condensate Sub-Balance

Declared from A-DC reference case:

V_condensate = 252 m³/day
R_condensate_capture = 0.90
V_condensate_captured = 227 m³/day
V_condensate_lost = 25 m³/day
10.9 W-env-DC Water Sub-Balance

Declared:

V_WCT_recovered = 132.3 m³/day
V_condensate_captured = 227 m³/day
V_external = 0 m³/day
V_rainwater = 0 m³/day
V_other_water = 0 m³/day

R_WENV = 0.90
V_WENV_bypass_clean = 0 m³/day

Input:

V_WENV_in =
132.3 + 227

V_WENV_in = 359.3 m³/day

Clean technical water:

V_clean_technical =
359.3 × 0.90

V_clean_technical = 323.37 m³/day

Reject:

V_WENV_reject =
359.3 - 323.37

V_WENV_reject = 35.93 m³/day
10.10 Final Water Balance

Final liquid reject:

V_final_liquid_reject =
V_WCT_waste + V_WENV_reject

V_final_liquid_reject =
44.1 + 35.93

V_final_liquid_reject = 80.03 m³/day

Fresh water reduction:

fresh_water_reduction =
min(323.37, 529.2)

fresh_water_reduction = 323.37 m³/day

New external make-up requirement:

V_external_makeup_new =
529.2 - 323.37

V_external_makeup_new = 205.83 m³/day

Make-up replacement fraction:

makeup_replacement_fraction =
323.37 / 529.2

makeup_replacement_fraction = 0.611

Liquid reject reduction relative to original blowdown:

liquid_reject_reduction =
V_blowdown_original - V_final_liquid_reject

liquid_reject_reduction =
176.4 - 80.03

liquid_reject_reduction = 96.37 m³/day
liquid_reject_reduction_fraction =
96.37 / 176.4

liquid_reject_reduction_fraction = 0.546

Boundary note:

V_final_liquid_reject includes W-env-DC reject generated from both W-CT recovered water and A-DC condensate.
Therefore, liquid reject reduction must be reported with boundary definition.
10.11 Electric Balance

Known declared loads:

P_ADC_fan = 213.7 kW
P_export_electric = 4.46 kW
P_aux_internal_load = 50 kW

Reference-case interpretation:

In this reference subtotal, P_aux_internal_load represents additional known SPHERE-DC support loads not already counted as P_ADC_fan or P_export_electric.

It must not include the same A-DC fan load or export pump/control load already listed above.
Known subtotal:

P_known_module_loads =
213.7 + 4.46 + 50

P_known_module_loads = 268.16 kW

Auxiliary gross:

P_aux_gross = 6.04 kW

Known external electricity required:

P_external_SPHERE_required_known =
max(0, 268.16 - 6.04)

P_external_SPHERE_required_known = 262.12 kW

Daily known electricity:

E_external_SPHERE_required_known_day =
262.12 × 24

E_external_SPHERE_required_known_day = 6290.9 kWh/day

Missing loads:

P_WCT_electric = site-specific
P_WENV_electric = site-specific
P_buffer_electric = site-specific
P_other_SPHERE_loads = site-specific

Interpretation:

The 10 MW integrated reference is not electrically self-sufficient.
Auxiliary power offsets only a small fraction of known support loads.
11. Comparative Summary Table
11.1 Heat
Metric	1 MW Case	10 MW Case
Q_total	1,000 kW	10,000 kW
Q_captured	630 kW	6,300 kW
Q_uncaptured_tail	370 kW	3,700 kW
Q_WCT	200 kW	2,000 kW
Q_ADC	150 kW	1,500 kW
Q_WENV	100 kW	1,000 kW
Q_export_candidate	120 kW	1,200 kW
Q_export_useful	113.45 kW	1,136.47 kW
Q_buffer_charge	30 kW	300 kW
Q_aux_used	30 kW	300 kW
P_aux_gross	0.604 kW_el	6.04 kW_el
Q_aux_residual_total	29.396 kW	293.96 kW
Q_tail_immediate	405.396 kW	4,053.96 kW
11.2 Water
Metric	1 MW Case	10 MW Case
V_makeup_original	52.95 m³/day	529.2 m³/day
V_blowdown_original	17.65 m³/day	176.4 m³/day
V_WCT_recovered	13.24 m³/day	132.3 m³/day
V_WCT_waste	4.41 m³/day	44.1 m³/day
V_condensate_captured	22.7 m³/day	227 m³/day
V_WENV_in	35.94 m³/day	359.3 m³/day
V_clean_technical	32.35 m³/day	323.37 m³/day
V_WENV_reject	3.59 m³/day	35.93 m³/day
V_final_liquid_reject	8.00 m³/day	80.03 m³/day
fresh_water_reduction	32.35 m³/day	323.37 m³/day
makeup_replacement_fraction	61.1%	61.1%
liquid_reject_reduction_fraction	54.7%	54.6%
11.3 Electric
Metric	1 MW Case	10 MW Case
P_ADC_fan	21.4 kW	213.7 kW
P_export_electric	0.676 kW	4.46 kW
P_aux_internal_load	7.5 kW	50 kW
P_known_module_loads	29.576 kW	268.16 kW
P_aux_gross	0.604 kW	6.04 kW
P_external_SPHERE_required_known	28.972 kW	262.12 kW
E_external_SPHERE_required_known_day	695.3 kWh/day	6,290.9 kWh/day
12. Integrated Consistency Checks
12.1 Heat Allocation Check
heat_allocation_valid =
(
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct
)
<= Q_captured

If false:

error = "captured heat overallocated"
12.2 Water Double-Counting Check
V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

Validation:

W-CT recovered water must not be counted as clean water until W-env-DC treatment or validation.

A-DC condensate must not be counted as clean water until W-env-DC treatment or validation.

The same m³ of water must not appear twice in V_clean_technical.
12.3 Electric Double-Counting Check
For each electric load:
    assign to exactly one category.

Categories:

P_WCT_electric
P_ADC_electric
P_WENV_electric
P_export_electric
P_buffer_electric
P_controls_total
P_aux_internal_load
P_other_SPHERE_loads

Validation:

If P_aux_internal_load already includes pumps and fans from modules,
do not add those module pump and fan loads again.
12.4 Thermal Tail Check
Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

Validation:

Immediate thermal tail must not be reported as zero unless every term is zero and all allocated heat has defined useful routing.
13. Sensitivity Variables

The integrated result is sensitive to:

eta_capture
eta_HX
Q_WCT
Q_ADC
Q_WENV
Q_export_demand
T_export_supply
Q_buffer_charge
E_buffer_required
Q_aux_used
T_hot_aux_C
T_cold_aux_C
eta_aux_conversion
V_blowdown_original
R_WCT
V_condensate_captured
R_WENV
P_ADC_fan
P_WCT_electric
P_WENV_electric

Dominant risks:

A-DC condensate depends strongly on climate and humidity.

W-CT recovery depends strongly on water chemistry and recovery rate.

W-env-DC clean water depends on treatment quality and reject handling.

Heat export depends on real external demand and temperature suitability.

Auxiliary power is weak under low-grade heat conditions.

Thermal buffers reduce immediate peaks but do not eliminate heat over full cycle.
14. AI Validation Checklist

An AI analysis system should verify:

1. Is Q_total equal to declared IT load plus declared facility extras?

2. Is Q_captured calculated from eta_capture and eta_HX?

3. Is Q_uncaptured_tail explicitly calculated?

4. Does captured heat allocation sum to Q_captured or less?

5. Is any heat overallocated?

6. Is heat export limited by real demand and interface capacity?

7. Is exported useful heat calculated after heat exchanger and pipe losses?

8. Is thermal buffer charge separated from permanent heat elimination?

9. Is auxiliary conversion limited by Carnot and practical efficiency?

10. Is auxiliary residual heat routed?

11. Is W-CT recovery calculated from blowdown and R_WCT?

12. Is W-CT waste explicitly retained?

13. Is A-DC condensate climate-dependent and not treated as free water?

14. Is W-env-DC input equal to W-CT recovery plus A-DC condensate plus external streams?

15. Is W-env-DC reject explicitly calculated?

16. Is clean technical water limited by treatment recovery and validation?

17. Is fresh water reduction limited by actual make-up demand?

18. Is final liquid reject calculated?

19. Are electric loads counted once only?

20. Is auxiliary power prevented from being claimed as data center power?

21. Are unknown module electric loads left as placeholders instead of silently ignored?

22. Are 1 MW and 10 MW reference cases scaled consistently?

23. Are all assumptions declared?

24. Are non-claim rules preserved?

25. Does the integrated balance avoid zero-heat, zero-water, and self-powered claims?
15. Minimum Site Data Required for Real Design
hourly IT load profile
PUE or facility overhead profile
cooling architecture
coolant supply temperature
coolant return temperature
available heat capture points
heat exchanger limits
cooling tower circulation
cooling tower evaporation
cooling tower blowdown
cycles of concentration
water chemistry
make-up water quality
blowdown quality
local air temperature profile
local humidity profile
air quality profile
A-DC airflow source
condensate quality
W-env-DC target water quality
external heat demand profile
external heat required temperature
heat export distance
thermal buffer volume constraints
auxiliary conversion technology
hot-side temperature profile
cold-side temperature profile
all pump loads
all fan loads
all treatment electric loads
all control electric loads
reject storage capacity
reject disposal route
fallback heat rejection capacity
BMS / SCADA integration points
legal water reuse constraints
legal heat export constraints
16. Non-Claim Rules for Integrated Balance

Not allowed:

SPHERE-DC eliminates data center heat.

SPHERE-DC creates zero thermal tail.

SPHERE-DC creates zero liquid discharge.

SPHERE-DC produces potable water.

SPHERE-DC is self-powered.

SPHERE-DC powers the data center from its own waste heat.

SPHERE-DC guarantees economic payback.

SPHERE-DC guarantees environmental benefit without site data.

SPHERE-DC clean water is valid without testing.

SPHERE-DC exported heat is useful without real demand.

SPHERE-DC buffer permanently stores heat.

Allowed claim format:

Under declared assumptions:
SPHERE-DC captures X kW of heat from Y kW IT load.
SPHERE-DC allocates heat to W-CT, A-DC, W-env-DC, heat export, buffer, and auxiliary conversion.
Immediate local thermal tail is Z kW under the declared routing model.
SPHERE-DC recovers A m³/day of clean technical water after treatment or validation.
Final liquid reject is B m³/day.
Fresh water make-up demand is reduced by C m³/day.
Auxiliary power produces D kW and offsets part of support loads only.
Unknown site-specific loads and quality constraints remain unresolved.
17. Summary Formula Block
Q_total =
P_IT + P_facility_extra

Q_captured =
Q_total × eta_capture × eta_HX

Q_uncaptured_tail =
Q_total - Q_captured

Q_captured =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Q_tail_captured_direct =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export_candidate
- Q_buffer_charge
- Q_aux_used

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

V_WCT_recovered =
V_blowdown_original × R_WCT

V_WCT_waste =
V_blowdown_original - V_WCT_recovered

V_condensate_captured =
V_condensate × R_condensate_capture

V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

V_WENV_treated_clean =
V_WENV_treatment_in × R_WENV

V_WENV_reject =
V_WENV_treatment_in - V_WENV_treated_clean

V_clean_technical =
V_WENV_treated_clean + V_WENV_bypass_clean

V_final_liquid_reject =
V_WCT_waste
+ V_WENV_reject
+ V_other_reject

fresh_water_reduction =
min(V_clean_technical, V_makeup_original)

V_external_makeup_new =
V_makeup_original - fresh_water_reduction

Q_export_candidate =
min(
    Q_export_available,
    Q_export_demand,
    Q_export_interface_max
)

Q_export_delivered =
max(0, Q_after_HX - Q_loss_pipe)

Q_export_useful =
min(Q_export_delivered, Q_export_demand)

P_aux_gross =
Q_aux_used × eta_aux_conversion × eta_power_electronics

P_aux_net =
P_aux_gross - P_aux_internal_load

P_external_SPHERE_required =
max(0, P_known_module_loads - P_aux_gross)
