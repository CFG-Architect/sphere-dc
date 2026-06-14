# w-env-dc.md

# SPHERE-DC Technical Document 05: W-env-DC Module

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
W-env-DC = Water / Environmental / Data Center Clean Technical Water Module

Concept baseline:

SPHERE-DC receives water streams from A-DC condensate, W-CT recovered water, and external water sources.
W-env-DC treats, validates, and routes these streams into clean technical water for cooling system make-up, SPHERE internal use, irrigation, or other local technical use.
W-env-DC must not count any input water as clean technical water until treatment or quality validation is complete.

W-env-DC is the SPHERE-DC module for treating, validating, and routing recovered water streams into clean technical water for cooling make-up, SPHERE internal use, and permitted local non-potable technical use.
```

## 1. Scope

This document defines calculation logic for:

- water input aggregation
- water quality classification
- treatment recovery
- clean technical water production
- reject stream calculation
- contaminant mass balance
- cooling make-up substitution
- SPHERE internal water demand
- irrigation / local technical demand
- heat demand of W-env-DC
- electric demand of W-env-DC
- water reuse efficiency
- failure and bypass conditions

This document does not define:

- final membrane selection
- final desalination equipment
- potable water certification
- municipal water permitting
- hazardous waste classification
- detailed chemical dosing
- final microbiological validation
- final equipment sizing

## 2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

W-env-DC priority condition:

Q_WENV is allocated after Q_WCT and Q_ADC, and before Q_export and Q_buffer.

Water routing priority:

1. preserve data center cooling safety
2. replace cooling tower make-up water
3. support SPHERE internal process water
4. support local technical / irrigation demand
5. route surplus or off-spec water to storage, retreatment, or reject

## 3. Required Inputs

### 3.1 Water Volume Inputs

Symbol	Meaning	Unit
V_WCT_recovered	Recovered water from W-CT	m³/day
V_condensate_captured	Captured condensate from A-DC	m³/day
V_external	External water input	m³/day
V_rainwater	Rainwater input, if available	m³/day
V_other_water	Other technical water input	m³/day
V_WENV_in	Total W-env-DC inlet water	m³/day
V_WENV_bypass_clean	Input water bypassed after validation as already suitable	m³/day
V_WENV_treatment_in	Water routed through treatment	m³/day

### 3.2 Water Demand Inputs

Symbol	Meaning	Unit
V_makeup_demand	Cooling system make-up water demand	m³/day
V_SPHERE_internal_demand	SPHERE internal technical water demand	m³/day
V_irrigation_demand	Permitted non-potable local use demand	m³/day
V_local_technical_demand	Other local technical water demand	m³/day
V_total_reuse_demand	Total useful water demand	m³/day

### 3.3 Treatment Inputs

Symbol	Meaning	Unit
R_WENV	W-env-DC treatment recovery rate	0–1
V_clean_technical	Clean technical water produced	m³/day
V_WENV_reject	Reject / brine / sludge stream	m³/day
V_WENV_loss	Process losses not included in reject	m³/day
Q_WENV_available	Heat available to W-env-DC	kW
Q_WENV_required	Heat required by W-env-DC	kW
Q_WENV_used	Heat used by W-env-DC	kW
P_WENV_electric	W-env-DC electric load	kW
SEC_WENV	Specific electric consumption	kWh/m³
STC_WENV	Specific thermal consumption	kWh_th/m³

### 3.4 Water Quality Inputs

Symbol	Meaning	Unit
TDS_in	Total dissolved solids at inlet	mg/L
TDS_out	TDS in treated water	mg/L
TDS_reject	TDS in reject stream	mg/L
TSS_in	Total suspended solids at inlet	mg/L
TSS_out	Total suspended solids at outlet	mg/L
pH_in	Inlet pH	pH
pH_out	Outlet pH	pH
EC_in	Inlet conductivity	µS/cm
EC_out	Outlet conductivity	µS/cm
hardness_in	Inlet hardness	mg/L as CaCO₃
hardness_out	Outlet hardness	mg/L as CaCO₃
metals_in	Metals concentration at inlet	mg/L
metals_out	Metals concentration at outlet	mg/L
biological_risk_in	Biological contamination risk indicator	boolean / score
quality_target	Required quality class for selected reuse	class / threshold set

## 4. Default Constants

rho_water = 1000 kg/m³
Cp_water = 4.186 kJ/kg·K
h_fg_water_default = 2450 kJ/kg
seconds_per_day = 86400 s/day
liters_per_m3 = 1000 L/m³
kWh_per_kJ = 1 / 3600

## 5. W-env-DC Input Balance

### 5.1 Total Inlet Water

V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

Constraint:

V_WENV_in >= 0

### 5.2 Treatment Inlet

If no input is validated as already suitable:

V_WENV_treatment_in = V_WENV_in

If some input is already validated as suitable for direct technical reuse:

V_WENV_treatment_in = V_WENV_in - V_WENV_bypass_clean

Constraints:

0 <= V_WENV_bypass_clean <= V_WENV_in
V_WENV_treatment_in >= 0

Non-double-count rule:

Water bypassed as already suitable must be counted once in total usable water.
Do not count the same water as both WCT recovery and WENV output unless WENV actually treats or validates it.

## 6. Treatment Recovery Calculation

### 6.1 Treated Clean Technical Water

V_WENV_treated_clean = V_WENV_treatment_in × R_WENV

### 6.2 W-env-DC Reject Stream

V_WENV_reject = V_WENV_treatment_in - V_WENV_treated_clean - V_WENV_loss

If process losses are not separately modeled:

V_WENV_reject = V_WENV_treatment_in × (1 - R_WENV)

Constraints:

0 <= R_WENV < 1
V_WENV_treated_clean <= V_WENV_treatment_in
V_WENV_reject >= 0

Non-claim rule:

R_WENV = 1.0 is not allowed in concept claims.
100% clean water recovery is not allowed.

### 6.3 Total Clean Technical Water Available

V_clean_technical =
V_WENV_treated_clean
+ V_WENV_bypass_clean

Constraint:

V_clean_technical <= V_WENV_in

## 7. Contaminant Mass Balance

For any component X:

M_X_in_kg_day = V_WENV_treatment_in × C_X_in / 1000

Where:

V_WENV_treatment_in in m³/day
C_X_in in mg/L
M_X_in_kg_day in kg/day

Treated water load:

M_X_clean_kg_day = V_WENV_treated_clean × C_X_out / 1000

Reject load:

M_X_reject_kg_day = M_X_in_kg_day - M_X_clean_kg_day

Reject concentration:

C_X_reject = (M_X_reject_kg_day × 1000) / V_WENV_reject

Constraints:

M_X_reject_kg_day >= 0
V_WENV_reject > 0 if C_X_reject is calculated
C_X_out must satisfy selected quality_target before reuse

## 8. Quality Target Logic

W-env-DC must classify output by target use.

Example classes:

Class T1: cooling tower make-up water
Class T2: SPHERE internal technical water
Class T3: permitted non-potable local technical use
Class T4: local non-potable technical use
Potable-water use is outside the SPHERE-DC baseline scope.

Generic quality check:

if water_quality_out <= quality_target_limits:
    water_status = "valid_for_target_use"
else:
    water_status = "requires_retreatment_or_reject"

Direct reuse condition:

Direct reuse is allowed only if:
measured_water_quality <= required_reuse_quality_limits

Potable-adjacent restriction:

Potable-water use requires a separate project, separate validation, separate regulatory approval, and must not be inferred from this repository.

## 9. Useful Water Allocation

### 9.1 Total Reuse Demand

V_total_reuse_demand =
V_makeup_demand
+ V_SPHERE_internal_demand
+ V_irrigation_demand
+ V_local_technical_demand

### 9.2 Allocation Priority

V_remaining_clean = V_clean_technical

V_to_makeup = min(V_remaining_clean, V_makeup_demand)
V_remaining_clean = V_remaining_clean - V_to_makeup

V_to_SPHERE_internal = min(V_remaining_clean, V_SPHERE_internal_demand)
V_remaining_clean = V_remaining_clean - V_to_SPHERE_internal

V_to_irrigation = min(V_remaining_clean, V_irrigation_demand)
V_remaining_clean = V_remaining_clean - V_to_irrigation

V_to_local_technical = min(V_remaining_clean, V_local_technical_demand)
V_remaining_clean = V_remaining_clean - V_to_local_technical

V_clean_surplus = V_remaining_clean

Constraints:

V_to_makeup <= V_makeup_demand
V_to_SPHERE_internal <= V_SPHERE_internal_demand
V_to_irrigation <= V_irrigation_demand
V_to_local_technical <= V_local_technical_demand
V_clean_surplus >= 0

## 10. Fresh Water Reduction

If W-env-DC output replaces external make-up water:

fresh_water_reduction_WENV = V_to_makeup

If it also replaces other external technical water:

fresh_water_reduction_total =
V_to_makeup
+ V_to_SPHERE_internal_external_substitution
+ V_to_irrigation_external_substitution
+ V_to_local_technical_external_substitution

Constraint:

fresh_water_reduction_total <= original_external_water_demand

## 11. Water Loop Closure Ratio

Define useful reused water:

V_reused_useful =
V_to_makeup
+ V_to_SPHERE_internal
+ V_to_irrigation
+ V_to_local_technical

Water loop closure ratio:

water_loop_closure_ratio =
V_reused_useful / V_total_reuse_demand

Constraint:

0 <= water_loop_closure_ratio <= 1

Alternative relative to W-env-DC input:

WENV_useful_recovery_ratio =
V_reused_useful / V_WENV_in

Constraint:

0 <= WENV_useful_recovery_ratio <= 1

## 12. Thermal Demand Model

W-env-DC may use heat for:

preheating
membrane support
distillation
desalination
thermal concentration
sanitization
regeneration
sludge drying

### 12.1 Thermal Evaporation / Distillation Case

If clean water is produced by evaporation / condensation:

m_evap_WENV_kg_s =
(V_WENV_treated_clean × rho_water) / seconds_per_day
Q_WENV_required =
(m_evap_WENV_kg_s × h_fg_effective) / eta_WENV_thermal_process

Where:

Q_WENV_required in kW
h_fg_effective in kJ/kg
eta_WENV_thermal_process in 0–1

### 12.2 Low-Thermal / Membrane-Support Case

If W-env-DC uses primarily membrane or hybrid treatment:

Q_WENV_required =
V_WENV_treatment_in × STC_WENV / 24

Where:

STC_WENV in kWh_th/m³
V_WENV_treatment_in in m³/day
Q_WENV_required in kW_th

### 12.3 Available Heat Constraint

Q_WENV_used = min(Q_WENV_available, Q_WENV_required)

If:

Q_WENV_available < Q_WENV_required

Then one or more must occur:

reduce treatment throughput
reduce recovery rate
use auxiliary heat
switch to lower-thermal process
route water to storage
route water to external treatment

Important distinction:

Q_WENV_required depends on the selected treatment technology.

Pure thermal evaporation / distillation may require much more heat than the integrated SPHERE-DC allocation provides.

Low-thermal, membrane-support, hybrid, staged, heat-recovery, or partial-treatment configurations may require lower direct thermal input.

Therefore:

Q_WENV_required is a technology-dependent process demand.

Q_WENV_available and Q_WENV_used represent heat allocated from the captured SPHERE thermal core to W-env-DC.

Q_WENV in the integrated balance must not be interpreted as proof that the full declared clean-water output can be produced by pure thermal evaporation.

If Q_WENV_required exceeds Q_WENV_available, the unresolved demand must be handled by reduced throughput, lower recovery, heat recovery, auxiliary heat, lower-thermal treatment, external treatment, storage, or bypass.

## 13. Electric Demand Model

Total electric demand:

P_WENV_electric =
P_pumps_WENV
+ P_controls_WENV
+ P_filtration_WENV
+ P_membrane_WENV
+ P_UV_or_sanitization_WENV
+ P_sludge_or_reject_handling_WENV

Specific electric consumption:

SEC_WENV =
P_WENV_electric × 24 / V_clean_technical

Where:

SEC_WENV in kWh/m³
P_WENV_electric in kW
V_clean_technical in m³/day

Constraint:

V_clean_technical > 0

## 14. Reference Case A: 1 MW Data Center

### 14.1 Declared Inputs

From previous module reference cases:

V_WCT_recovered = 13.24 m³/day
V_condensate_captured = 22.7 m³/day
V_external = 0 m³/day
V_rainwater = 0 m³/day
V_other_water = 0 m³/day
R_WENV = 0.90
V_WENV_bypass_clean = 0 m³/day

Demand assumptions:

V_makeup_demand = 52.95 m³/day
V_SPHERE_internal_demand = 2.0 m³/day
V_irrigation_demand = 5.0 m³/day
V_local_technical_demand = 0 m³/day

### 14.2 Input Balance

V_WENV_in =
13.24 + 22.7 + 0 + 0 + 0

V_WENV_in = 35.94 m³/day
V_WENV_treatment_in = 35.94 m³/day

### 14.3 Clean Water and Reject

V_WENV_treated_clean = 35.94 × 0.90
V_WENV_treated_clean = 32.35 m³/day
V_WENV_reject = 35.94 - 32.35
V_WENV_reject = 3.59 m³/day
V_clean_technical = 32.35 m³/day

### 14.4 Useful Water Allocation

V_remaining_clean = 32.35

V_to_makeup = min(32.35, 52.95)
V_to_makeup = 32.35

V_remaining_clean = 0

V_to_SPHERE_internal = 0
V_to_irrigation = 0
V_to_local_technical = 0
V_clean_surplus = 0

### 14.5 Fresh Water Reduction

fresh_water_reduction_WENV = 32.35 m³/day

Relative to make-up demand:

makeup_replacement_fraction =
32.35 / 52.95

makeup_replacement_fraction = 0.611

Result:

Under declared assumptions, W-env-DC can replace 61.1% of cooling make-up water in the 1 MW reference case.
Remaining W-env-DC reject = 3.59 m³/day.

### 14.6 Thermal Evaporation Sensitivity

If W-env-DC output is produced by thermal evaporation:

m_evap_WENV =
(32.35 × 1000) / 86400

m_evap_WENV = 0.374 kg/s

Assume:

h_fg_effective = 2450 kJ/kg
eta_WENV_thermal_process = 1.0
Q_WENV_required =
0.374 × 2450

Q_WENV_required = 916 kW

Interpretation:

Pure thermal evaporation would require ~916 kW for 32.35 m³/day.
This is large relative to a 1 MW data center and requires heat recovery, multi-effect design, lower-energy treatment, or lower throughput.

## 15. Reference Case B: 10 MW Data Center

### 15.1 Declared Inputs

From previous module reference cases:

V_WCT_recovered = 132.3 m³/day
V_condensate_captured = 227 m³/day
V_external = 0 m³/day
V_rainwater = 0 m³/day
V_other_water = 0 m³/day
R_WENV = 0.90
V_WENV_bypass_clean = 0 m³/day

Demand assumptions:

V_makeup_demand = 529.2 m³/day
V_SPHERE_internal_demand = 20 m³/day
V_irrigation_demand = 50 m³/day
V_local_technical_demand = 0 m³/day

### 15.2 Input Balance

V_WENV_in =
132.3 + 227 + 0 + 0 + 0

V_WENV_in = 359.3 m³/day
V_WENV_treatment_in = 359.3 m³/day

### 15.3 Clean Water and Reject

V_WENV_treated_clean = 359.3 × 0.90
V_WENV_treated_clean = 323.37 m³/day
V_WENV_reject = 359.3 - 323.37
V_WENV_reject = 35.93 m³/day
V_clean_technical = 323.37 m³/day

### 15.4 Useful Water Allocation

V_remaining_clean = 323.37

V_to_makeup = min(323.37, 529.2)
V_to_makeup = 323.37

V_remaining_clean = 0

V_to_SPHERE_internal = 0
V_to_irrigation = 0
V_to_local_technical = 0
V_clean_surplus = 0

### 15.5 Fresh Water Reduction

fresh_water_reduction_WENV = 323.37 m³/day

Relative to make-up demand:

makeup_replacement_fraction =
323.37 / 529.2

makeup_replacement_fraction = 0.611

Result:

Under declared assumptions, W-env-DC can replace 61.1% of cooling make-up water in the 10 MW reference case.
Remaining W-env-DC reject = 35.93 m³/day.

### 15.6 Thermal Evaporation Sensitivity

If W-env-DC output is produced by thermal evaporation:

m_evap_WENV =
(323.37 × 1000) / 86400

m_evap_WENV = 3.743 kg/s

Assume:

h_fg_effective = 2450 kJ/kg
eta_WENV_thermal_process = 1.0
Q_WENV_required =
3.743 × 2450

Q_WENV_required = 9170 kW

Interpretation:

Pure thermal evaporation would require ~9.17 MW for 323.37 m³/day.
This is too large to treat as a free secondary process in a 10 MW data center.
Hybrid treatment, heat recovery, multi-effect design, or reduced throughput must be modeled explicitly.

## 16. TDS Example Mass Balance

Example assumptions for 1 MW reference case:

V_WENV_treatment_in = 35.94 m³/day
R_WENV = 0.90
V_WENV_treated_clean = 32.35 m³/day
V_WENV_reject = 3.59 m³/day

TDS_in = 1000 mg/L
TDS_out = 250 mg/L

Input TDS mass:

M_TDS_in =
35.94 × 1000 / 1000

M_TDS_in = 35.94 kg/day

Clean water TDS mass:

M_TDS_clean =
32.35 × 250 / 1000

M_TDS_clean = 8.09 kg/day

Reject TDS mass:

M_TDS_reject =
35.94 - 8.09

M_TDS_reject = 27.85 kg/day

Reject TDS concentration:

TDS_reject =
(27.85 × 1000) / 3.59

TDS_reject = 7758 mg/L

Interpretation:

High recovery concentrates contaminants.
Reject volume decreases, but reject concentration increases.
Reject handling must be explicit.

## 17. Process Output Requirements

W-env-DC calculation must output:

V_WCT_recovered
V_condensate_captured
V_external
V_rainwater
V_other_water
V_WENV_in
V_WENV_bypass_clean
V_WENV_treatment_in
R_WENV
V_WENV_treated_clean
V_WENV_reject
V_clean_technical
V_to_makeup
V_to_SPHERE_internal
V_to_irrigation
V_to_local_technical
V_clean_surplus
fresh_water_reduction_WENV
water_loop_closure_ratio
WENV_useful_recovery_ratio
TDS_in
TDS_out
TDS_reject
M_TDS_in
M_TDS_clean
M_TDS_reject
Q_WENV_required
Q_WENV_available
Q_WENV_used
P_WENV_electric
SEC_WENV

## 18. Control Logic

INPUT:
    V_WCT_recovered
    V_condensate_captured
    V_external
    water_quality_by_stream
    quality_target_by_use
    V_makeup_demand
    V_SPHERE_internal_demand
    V_irrigation_demand
    Q_WENV_available
    R_WENV_target

PROCESS:
    aggregate inlet water streams
    classify water quality by source
    determine direct bypass eligibility
    calculate V_WENV_treatment_in
    calculate feasible R_WENV
    calculate V_clean_technical
    calculate V_WENV_reject
    calculate contaminant mass balance
    verify output water quality
    calculate heat demand
    compare Q_WENV_required with Q_WENV_available
    calculate electric demand
    allocate clean water by priority

IF output_quality_valid_for_makeup:
    route water to cooling make-up

ELSE IF output_quality_valid_for_SPHERE_internal:
    route water to SPHERE internal use

ELSE IF output_quality_valid_for_irrigation:
    route water to irrigation / local technical use

ELSE:
    route water to retreatment, storage, or reject

IF Q_WENV_available < Q_WENV_required:
    reduce throughput
    reduce R_WENV
    use auxiliary heat
    switch treatment mode
    or route water to storage / external treatment

OUTPUT:
    clean technical water
    reject stream
    updated fresh water demand
    updated water reuse ratio
    updated heat and electric demand

## 19. Failure / Bypass Conditions

IF WENV_fault == true:
    isolate W-env-DC
    preserve data center cooling make-up supply
    route incoming water to storage, W-CT fallback, or external treatment
    route Q_WENV allocation to next SPHERE priority module
IF output_water_quality_out_of_spec == true:
    block reuse
    route to retreatment or reject
IF reject_tank_full == true:
    stop treatment
    preserve safe inlet water storage or external discharge route
IF biological_risk_detected == true:
    block direct reuse
    sanitize system
    route to treatment or reject
IF cooling_makeup_demand_requires_water and WENV_unavailable:
    use original make-up water supply

## 20. Engineering Constraints

0 <= R_WENV < 1

V_clean_technical <= V_WENV_in

V_WENV_reject >= 0

V_WENV_treated_clean <= V_WENV_treatment_in

V_WENV_bypass_clean <= V_WENV_in

V_to_makeup <= V_makeup_demand

fresh_water_reduction_total <= original_external_water_demand

Q_WENV_used <= Q_WENV_available

P_WENV_electric must be included in system electric load.

Reject stream must be tracked.

Clean technical water must meet target quality before reuse.

W-env-DC must not interrupt data center cooling make-up safety.

W-env-DC must be bypassable.

No potable water claim is allowed without separate validation.

## 21. AI Validation Questions

An AI analysis system should verify:
1. Are all inlet water streams defined?
2. Is V_WENV_in equal to the sum of declared inputs?
3. Is any water stream double-counted?
4. Is R_WENV declared as an assumption?
5. Is R_WENV < 1?
6. Is V_clean_technical <= V_WENV_in?
7. Is V_WENV_reject explicitly calculated?
8. Are contaminants conserved across clean water and reject?
9. Is reject concentration calculated when recovery is high?
10. Is clean technical water quality specified before reuse?
11. Is condensate treated or validated before being counted as clean water?
12. Is W-CT recovered water treated or validated before reuse?
13. Is fresh water reduction limited by actual demand?
14. Is heat demand calculated if thermal purification is claimed?
15. Does Q_WENV_required exceed available captured heat?
16. Is electric demand included in system load?
17. Is potable water avoided as a baseline claim?
18. Is fallback routing defined if W-env-DC fails?
19. Are 1 MW and 10 MW cases scaled from declared assumptions?
20. Which assumptions dominate feasibility: R_WENV, input quality, condensate volume, W-CT recovery, heat demand, or reuse demand?

## 22. Minimum Site Data Required

V_WCT_recovered
quality of W-CT recovered water
V_condensate_captured
quality of A-DC condensate
external water volume
external water quality
rainwater availability
cooling make-up water demand
SPHERE internal water demand
irrigation / local technical demand
required cooling tower make-up quality
required technical water quality
TDS
TSS
EC
pH
hardness
chloride
silica
metals
biological contamination risk
biocide residual
corrosion inhibitor residual
existing water treatment system
available Q_WENV
available electric power
reject handling capacity
reject disposal cost
water storage capacity
BMS / SCADA integration points
local water price
wastewater discharge cost
regulatory constraints

## 23. Non-Claim Rules

Not allowed without site-specific proof:

100% clean water recovery
zero reject
zero water discharge
potable water production
free water production
clean condensate without testing
clean W-CT output without testing
universal replacement of city water
guaranteed closed water loop
guaranteed economic payback

Allowed claim format:

Under declared assumptions:
W-env-DC receives X m³/day of water.
W-env-DC produces Y m³/day of clean technical water.
W-env-DC creates Z m³/day of reject stream.
W-env-DC can replace up to A m³/day of cooling make-up water if quality targets are met.
Output water requires validation for each target use.

## 24. Summary Formula Block

V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

V_WENV_treatment_in =
V_WENV_in - V_WENV_bypass_clean

V_WENV_treated_clean =
V_WENV_treatment_in × R_WENV

V_WENV_reject =
V_WENV_treatment_in - V_WENV_treated_clean - V_WENV_loss

V_WENV_reject =
V_WENV_treatment_in × (1 - R_WENV)

V_clean_technical =
V_WENV_treated_clean + V_WENV_bypass_clean

M_X_in_kg_day =
V_WENV_treatment_in × C_X_in / 1000

M_X_clean_kg_day =
V_WENV_treated_clean × C_X_out / 1000

M_X_reject_kg_day =
M_X_in_kg_day - M_X_clean_kg_day

C_X_reject =
(M_X_reject_kg_day × 1000) / V_WENV_reject

V_total_reuse_demand =
V_makeup_demand
+ V_SPHERE_internal_demand
+ V_irrigation_demand
+ V_local_technical_demand

V_to_makeup =
min(V_clean_technical, V_makeup_demand)

fresh_water_reduction_WENV =
V_to_makeup

V_reused_useful =
V_to_makeup
+ V_to_SPHERE_internal
+ V_to_irrigation
+ V_to_local_technical

water_loop_closure_ratio =
V_reused_useful / V_total_reuse_demand

WENV_useful_recovery_ratio =
V_reused_useful / V_WENV_in

m_evap_WENV_kg_s =
(V_WENV_treated_clean × rho_water) / seconds_per_day

Q_WENV_required =
(m_evap_WENV_kg_s × h_fg_effective) / eta_WENV_thermal_process

Q_WENV_required =
V_WENV_treatment_in × STC_WENV / 24

Q_WENV_used =
min(Q_WENV_available, Q_WENV_required)

P_WENV_electric =
P_pumps_WENV
+ P_controls_WENV
+ P_filtration_WENV
+ P_membrane_WENV
+ P_UV_or_sanitization_WENV
+ P_sludge_or_reject_handling_WENV

SEC_WENV =
P_WENV_electric × 24 / V_clean_technical
