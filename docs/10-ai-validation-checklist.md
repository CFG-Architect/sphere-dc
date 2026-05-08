# ai-validation-checklist.md

# SPHERE-DC Technical Document 10: AI Validation Checklist

## 0. Document Type

AI-evaluable validation protocol.

Target system:

```text
SPHERE-DC = thermal-water-air-energy integration layer around a data center

Purpose:

This document defines how an AI analysis system should evaluate the SPHERE-DC technical package.

The validation goal is not to prove commercial readiness.
The validation goal is to detect:
- feasibility boundaries
- missing variables
- invalid assumptions
- unit errors
- double counting
- overclaims
- weak points
- required site data
- dominant sensitivity variables
```

1. Documents to Validate

The AI system should evaluate the following documents together:

01 thermal-core.md
02 assumptions-and-variables.md
03 w-ct.md
04 a-dc.md
05 w-env-dc.md
06 thermal-buffers.md
07 heat-export.md
08 auxiliary-power.md
09 integrated-balance-1mw-10mw.md

This document validates the package as one system.

2. Validation Output Format

The AI validator should return results in this structure:

1. Executive technical verdict
2. System-level feasibility status
3. Module-level validation table
4. Heat balance validation
5. Water balance validation
6. Electric balance validation
7. Site-data gaps
8. Overclaim / non-claim audit
9. Dominant sensitivity variables
10. Critical failure modes
11. Recommended next calculations
12. Final AI feasibility classification
3. Validation Status Labels

Use only these labels:

PASS = internally consistent under declared assumptions

WARN = plausible but depends on missing site-specific data

FAIL = internally inconsistent, physically invalid, or overclaimed

UNKNOWN = cannot evaluate because required data is missing

NOT_APPLICABLE = item does not apply to the declared case
4. Severity Labels

Use only these severity labels:

S0 = informational
S1 = minor issue
S2 = important assumption gap
S3 = major feasibility risk
S4 = critical invalidation risk
S5 = impossible / physically invalid / false claim
5. Core Validation Principles
1. Validate declared assumptions first.

2. Do not replace declared assumptions silently.

3. Check units before checking conclusions.

4. Check conservation of energy and mass.

5. Check whether heat, water, or electricity is counted twice.

6. Distinguish captured heat from useful heat.

7. Distinguish temporary heat storage from heat elimination.

8. Distinguish recovered water from clean technical water.

9. Distinguish auxiliary support power from data center power.

10. Distinguish useful heat export from external heat rejection.

11. Treat all reference cases as illustrative, not universal.

12. Reject any zero-heat, zero-discharge, self-powered, or potable-water claim unless explicitly proven.
6. Required High-Level Verdict

The AI validator must classify the entire package using one of the following:

CLASS_A = internally consistent concept model; ready for site-specific engineering evaluation

CLASS_B = partially consistent; needs missing site data before engineering evaluation

CLASS_C = conceptually plausible but current assumptions are too weak or too optimistic

CLASS_D = internally inconsistent; requires correction before further evaluation

CLASS_E = physically invalid or based on false claims

The validator must also provide:

primary_reason_for_classification
top_5_technical_risks
top_5_missing_site_data_items
top_5_most_sensitive_variables
7. Global Input Validation
7.1 Required Global Variables

Check that the following variables are declared:

P_IT
P_facility_extra
Q_total
eta_capture
eta_HX
Q_captured
Q_uncaptured_tail
Q_WCT
Q_ADC
Q_WENV
Q_export_candidate
Q_buffer_charge
Q_aux_used
Q_tail_captured_direct
Q_tail_immediate

Validation:

IF any required global variable is missing:
    status = UNKNOWN
    severity = S2 or S3
7.2 Unit Consistency

Check:

Power variables use kW or MW consistently.

Energy variables use kWh, MWh, kJ, or MJ consistently.

Temperature differences use K.

Celsius temperatures are not used in Carnot calculations without conversion to Kelvin.

Water volume uses m³/day, m³/h, or m³/s explicitly.

Mass flow uses kg/s.

Airflow uses m³/h or m³/s explicitly.

Water quality uses mg/L, ppm, µS/cm, or declared equivalent.

Efficiency variables are dimensionless 0–1, not percent unless explicitly converted.

Failure condition:

IF unit conversion changes the result by >5%:
    status = FAIL
    severity = S3 or higher
8. Thermal-Core Validation
8.1 Total Heat

Validate:

Q_total ≈ P_IT

or:

Q_total = P_IT + P_facility_extra

Checks:

P_IT >= 0
P_facility_extra >= 0
Q_total >= P_IT

Failure:

IF Q_total < P_IT:
    status = FAIL
    severity = S4
8.2 Captured Heat

Validate:

Q_captured = Q_total × eta_capture × eta_HX

Checks:

0 <= eta_capture <= 1
0 <= eta_HX <= 1
Q_captured <= Q_total

Failure:

IF Q_captured > Q_total:
    status = FAIL
    severity = S5
8.3 Core Flow Rate

Validate:

m_dot_core = Q_captured / (Cp_water × DeltaT_core)
V_dot_core_m3_s = m_dot_core / rho_water
V_dot_core_m3_h = V_dot_core_m3_s × 3600

Checks:

DeltaT_core > 0
Cp_water > 0
rho_water > 0
V_dot_core is compatible with site hydraulic limits
9. Captured Heat Allocation Validation

Validate:

Q_allocated =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Required condition:

Q_allocated <= Q_captured

If:

Q_allocated > Q_captured

Then:

status = FAIL
severity = S5
error = "captured heat overallocated"

If:

Q_allocated < Q_captured

Then:

Q_tail_captured_direct =
Q_captured - Q_allocated

Validation:

The missing captured heat must be assigned to Q_tail_captured_direct or another explicit route.
10. Thermal Tail Validation

Validate:

Q_uncaptured_tail = Q_total - Q_captured

Validate integrated immediate tail:

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

Checks:

Q_tail_immediate >= 0
Q_uncaptured_tail >= 0
Q_tail_captured_direct >= 0

Failure:

IF Q_tail_immediate < 0:
    status = FAIL
    severity = S5

Non-claim check:

IF Q_tail_immediate is claimed as zero:
    verify every heat output route and residual term.
11. W-CT Validation
11.1 Cooling Tower Balance

Validate:

m_evap_kg_s = Q_CT / h_fg_water_evap_default
V_evap_m3_day = (m_evap_kg_s / rho_water) × 86400
V_blowdown = V_evap / (CoC - 1)
V_makeup = V_evap + V_blowdown + V_drift + V_leak

Checks:

Q_CT >= 0
CoC > 1
V_evap >= 0
V_blowdown >= 0
V_makeup >= V_evap

Failure:

IF CoC <= 1:
    status = FAIL
    severity = S5
11.2 W-CT Recovery

Validate:

V_WCT_recovered = V_blowdown × R_WCT
V_WCT_waste = V_blowdown - V_WCT_recovered

Checks:

0 <= R_WCT < 1
V_WCT_recovered <= V_blowdown
V_WCT_waste > 0

Failure:

IF R_WCT = 1:
    status = FAIL
    severity = S4
    issue = "100% W-CT recovery claim"
11.3 Contaminant Mass Balance

For each contaminant X, validate:

M_X_in = V_WCT_in × C_X_in / 1000
M_X_recovered = V_WCT_recovered × C_X_recovered / 1000
M_X_waste = M_X_in - M_X_recovered

Checks:

M_X_in >= 0
M_X_recovered >= 0
M_X_waste >= 0
C_X_waste is calculated if waste concentration is relevant

Failure:

IF contaminants disappear without reject or treatment mechanism:
    status = FAIL
    severity = S4
12. A-DC Validation
12.1 Airflow

Validate:

V_dot_air_m3_s = V_dot_air / 3600
m_dot_air = rho_air × V_dot_air_m3_s

Checks:

V_dot_air > 0
rho_air > 0
m_dot_air > 0
12.2 Humidity and Condensate

Validate:

m_dot_condensate = m_dot_air × max(0, omega_in - omega_out)
V_condensate = (m_dot_condensate / rho_water) × 86400

Checks:

omega_in >= 0
omega_out >= 0
condensate is claimed only if omega_out < omega_in

Failure:

IF condensate is claimed when omega_out >= omega_in:
    status = FAIL
    severity = S5
12.3 Latent Load

Validate:

Q_latent_ADC = m_dot_condensate × h_fg_water

Required condition:

If condensate is claimed, latent thermal load must be included.

Failure:

IF V_condensate > 0 and Q_latent_ADC is omitted:
    status = FAIL
    severity = S4
12.4 Fan Power

Validate:

P_fan = (DeltaP_total_ADC × V_dot_air_m3_s) / eta_fan / 1000

Checks:

DeltaP_total_ADC >= 0
0 < eta_fan <= 1
P_fan >= 0
12.5 Filtration

Validate:

eta_filter_PM = (PM_in - PM_out) / PM_in
M_PM_removed =
(PM_in - PM_out) × V_dot_air × 24 × 1e-9

Checks:

PM_in > 0
PM_out >= 0
PM_out <= PM_in
0 <= eta_filter_PM <= 1
13. W-env-DC Validation
13.1 Inlet Balance

Validate:

V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

Checks:

All streams are non-negative.
No stream is counted twice.
Recovered water and condensate are not counted as clean before treatment or validation.
13.2 Treatment Recovery

Validate:

V_WENV_treated_clean =
V_WENV_treatment_in × R_WENV
V_WENV_reject =
V_WENV_treatment_in - V_WENV_treated_clean - V_WENV_loss

If no separate loss:

V_WENV_reject =
V_WENV_treatment_in × (1 - R_WENV)

Checks:

0 <= R_WENV < 1
V_WENV_treated_clean <= V_WENV_treatment_in
V_WENV_reject >= 0

Failure:

IF R_WENV = 1:
    status = FAIL
    severity = S4
    issue = "100% W-env-DC recovery claim"
13.3 Clean Technical Water

Validate:

V_clean_technical =
V_WENV_treated_clean + V_WENV_bypass_clean

Checks:

V_clean_technical <= V_WENV_in
V_WENV_bypass_clean must be quality-validated

Failure:

IF water is labeled clean without treatment or validation:
    status = FAIL
    severity = S4
14. Final Water Balance Validation

Validate:

V_final_liquid_reject =
V_WCT_waste
+ V_WENV_reject
+ V_other_reject

Validate fresh water reduction:

fresh_water_reduction =
min(V_clean_technical, V_makeup_original)

Validate new make-up demand:

V_external_makeup_new =
V_makeup_original - fresh_water_reduction

Checks:

V_final_liquid_reject >= 0
fresh_water_reduction <= V_makeup_original
V_external_makeup_new >= 0

Failure:

IF zero liquid discharge is claimed while V_final_liquid_reject > 0:
    status = FAIL
    severity = S5
15. Thermal Buffers Validation
15.1 Peak Capacity

Validate:

E_peak = Q_peak_excess × t_peak
E_buffer_required = E_peak × safety_factor_buffer

Checks:

Q_peak_excess >= 0
t_peak >= 0
safety_factor_buffer >= 1
15.2 Water Buffer Sizing

Validate:

m_buffer_water =
(E_buffer_required × 3600) / (Cp_water × DeltaT_buffer)
V_buffer_water =
m_buffer_water / rho_water

Checks:

DeltaT_buffer > 0
Cp_water > 0
rho_water > 0
15.3 Buffer Logic

Required checks:

SOC_buffer is tracked.
SOC_min is declared.
SOC_max is declared.
Buffer full condition is defined.
Buffer empty condition is defined.
Thermal losses are included for storage-duration claims.

Failure:

IF stored heat is counted as eliminated:
    status = FAIL
    severity = S5
16. Heat Export Validation
16.1 Export Availability

Validate:

Q_export_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV

Validate candidate export:

Q_export_candidate =
min(
    Q_export_available,
    Q_export_demand,
    Q_export_interface_max
)

Checks:

Q_export_candidate <= Q_export_available
Q_export_candidate <= Q_export_demand
Q_export_candidate <= Q_export_interface_max
16.2 Temperature Suitability

Validate:

T_export_supply >= T_required_supply + T_margin_supply

Failure:

IF T_export_supply < T_required_supply:
    Q_export_useful = 0 unless temperature lift is modeled
16.3 Delivered and Useful Heat

Validate:

Q_export_delivered =
max(0, Q_after_HX - Q_loss_pipe)
Q_export_useful =
min(Q_export_delivered, Q_export_demand)

Checks:

Q_export_delivered <= Q_export_candidate
Q_export_useful <= Q_export_delivered
Q_export_useful <= Q_export_demand

Failure:

IF exported heat is counted as useful without real demand:
    status = FAIL
    severity = S5
16.4 Pump Power

Validate:

P_pump_export =
(DeltaP_export × V_dot_export_m3_s) / eta_pump_export / 1000

Checks:

DeltaP_export >= 0
0 < eta_pump_export <= 1
P_export_electric is included in system electric load
17. Auxiliary Power Validation
17.1 Temperature Conversion

Validate:

T_hot_aux_K = T_hot_aux_C + 273.15
T_cold_aux_K = T_cold_aux_C + 273.15

Checks:

T_hot_aux_K > T_cold_aux_K > 0

Failure:

IF Celsius is used directly in Carnot formula:
    status = FAIL
    severity = S4
17.2 Carnot Limit

Validate:

eta_Carnot_aux =
1 - (T_cold_aux_K / T_hot_aux_K)

Validate practical efficiency:

eta_aux_conversion =
eta_Carnot_aux × eta_practical_fraction

Checks:

0 < eta_Carnot_aux < 1
0 <= eta_practical_fraction <= 1
0 <= eta_aux_conversion < eta_Carnot_aux

Failure:

IF eta_aux_conversion >= eta_Carnot_aux:
    status = FAIL
    severity = S5
17.3 Gross and Net Power

Validate:

P_aux_gross =
Q_aux_used × eta_aux_conversion × eta_power_electronics

Validate:

P_aux_net =
P_aux_gross - P_aux_internal_load

Checks:

Q_aux_used <= Q_aux_available
P_aux_gross <= Q_aux_used
P_aux_internal_load is declared
P_aux_net is reported

Failure:

IF auxiliary power is claimed as data center power:
    status = FAIL
    severity = S5

Failure:

IF self-powered SPHERE is claimed while P_aux_net < 0:
    status = FAIL
    severity = S5
18. Integrated Electric Balance Validation

Validate:

P_known_module_loads =
P_WCT_electric
+ P_ADC_electric
+ P_WENV_electric
+ P_export_electric
+ P_buffer_electric
+ P_controls_total
+ P_other_SPHERE_loads

Validate:

P_external_SPHERE_required =
max(0, P_known_module_loads - P_aux_gross)

Checks:

Every pump load appears once.
Every fan load appears once.
Every control load appears once.
Every treatment load appears once.
Unknown loads remain explicit placeholders.

Failure:

IF unknown electric loads are omitted but conclusions are stated as complete:
    status = FAIL
    severity = S4
19. Double-Counting Audit
19.1 Heat Double Counting

Check that no heat stream appears in two useful categories simultaneously.

Invalid example:

The same 100 kW counted as:
- Q_export_useful
- Q_buffer_charge
- Q_aux_used

Validation rule:

Each kW of Q_captured must have exactly one immediate route.

Allowed later reuse:

Heat may move from buffer to later process only in time-series accounting.
It must not be counted twice in the same time step.
19.2 Water Double Counting

Invalid example:

The same condensate counted as:
- A-DC recovered water
- W-env-DC clean technical water
- cooling make-up replacement
without treatment or allocation sequence

Validation rule:

Each m³ of water must follow one traceable path:
source → treatment / validation → output → use / reject.
19.3 Electric Double Counting

Invalid example:

P_fan counted in:
- P_ADC_electric
- P_aux_internal_load
- P_known_module_loads
as separate loads

Validation rule:

Each electric load must appear exactly once in the integrated load total.
20. Non-Claim Audit

The AI validator must search for the following forbidden claims:

zero heat
zero thermal tail
heat elimination
self-powered data center
data center powered by its own waste heat
zero liquid discharge
100% water recovery
potable water
free water generation
clean condensate without treatment
clean W-CT output without treatment
free heat export
useful heat export without external demand
lossless thermal storage
permanent heat storage
guaranteed payback
guaranteed environmental benefit

For each detected claim:

claim_text
claim_location
status = PASS / WARN / FAIL
severity
required_proof
current_proof_available = true / false

Default rule:

If proof is not provided, classify forbidden claim as FAIL.
21. Sensitivity Analysis Requirements

The AI validator must identify dominant variables.

Minimum required sensitivity set:

eta_capture
eta_HX
DeltaT_core
Q_WCT
R_WCT
CoC
TDS_blowdown
Q_ADC
V_dot_air
RH_in
RH_out
DeltaP_total_ADC
R_condensate_capture
R_WENV
Q_WENV_required
Q_export_demand
T_export_supply
L_pipe
U_pipe
Q_buffer_charge
E_buffer_required
T_hot_aux_C
T_cold_aux_C
eta_practical_fraction
P_ADC_fan
P_WCT_electric
P_WENV_electric
P_known_module_loads

Required output:

top_10_sensitivity_variables
effect_direction
risk_level
data_needed_to_reduce_uncertainty

Example format:

Variable	Effect Direction	Risk Level	Required Data
RH_in - RH_out	controls condensate volume and latent load	high	hourly psychrometric profile
R_WCT	controls recovered water and reject concentration	high	pilot treatment data
T_export_supply	controls useful heat export eligibility	high	real temperature profile
eta_aux_conversion	controls support electricity	medium	technology-specific test data
22. Scenario Testing

The AI validator should run at least these scenarios:

Scenario 1: declared reference case

Scenario 2: low heat capture
eta_capture reduced by 25%

Scenario 3: low water recovery
R_WCT and R_WENV reduced by 25%

Scenario 4: dry climate
A-DC condensate reduced by 75%

Scenario 5: no external heat demand
Q_export_demand = 0

Scenario 6: buffer full
SOC_buffer = SOC_max

Scenario 7: auxiliary conversion unavailable
P_aux_gross = 0

Scenario 8: high fan pressure drop
DeltaP_total_ADC doubled

Scenario 9: W-env-DC output quality failure
V_clean_technical = 0 until retreatment

Scenario 10: heat export temperature mismatch
T_export_supply < T_required_supply

For each scenario, report:

Q_tail_immediate
V_clean_technical
V_final_liquid_reject
P_external_SPHERE_required
failed modules
dominant cause
23. Required Red-Flag Detection

The AI validator must flag these as major risks:

A-DC condensate volume depends on aggressive humidity targets.

A-DC latent load exceeds available thermal or cooling capacity.

W-CT recovery produces reject concentration without disposal route.

W-env-DC clean water depends on unvalidated water quality.

Heat export lacks real nearby heat demand.

Heat export temperature is too low for target user.

Thermal buffer is counted as permanent heat removal.

Auxiliary power is too small but described as self-powering.

Electric loads are incomplete.

Captured heat allocation exceeds Q_captured.

Final liquid reject is hidden or omitted.

Unknown site data is replaced by optimistic assumptions.

1 MW and 10 MW cases are scaled linearly without checking equipment limits.

Each red flag requires:

red_flag_name
affected_document
severity
why_it_matters
data_needed
possible_correction
24. Site Data Gap Audit

The AI validator must list missing site data in this structure:

data_item
required_for
current_status
impact_if_missing
priority

Priority levels:

P0 = required before any feasibility claim
P1 = required before engineering design
P2 = required before economic analysis
P3 = required before optimization

Minimum P0 data:

hourly IT load
cooling architecture
coolant supply/return temperatures
available heat capture points
cooling tower blowdown volume
cooling tower water chemistry
local hourly temperature and humidity
A-DC air stream definition
condensate quality
target water quality
external heat demand profile
external heat required temperature
SPHERE module electric loads
fallback heat rejection capacity
reject disposal route
25. Feasibility Classification Rules
25.1 CLASS_A

Assign CLASS_A only if:

all balances are internally consistent
no S4 or S5 failures
site-specific data is sufficient for preliminary engineering
non-claim rules are preserved
dominant risks are quantified
25.2 CLASS_B

Assign CLASS_B if:

internal logic is consistent
no impossible claims are made
some P0 or P1 site data is missing
reference cases are plausible but not site-validated
25.3 CLASS_C

Assign CLASS_C if:

concept is physically plausible
but key assumptions are optimistic or weak
dominant modules depend on unvalidated assumptions
site feasibility is uncertain
25.4 CLASS_D

Assign CLASS_D if:

the system has internal contradictions
heat, water, or electricity is double-counted
major calculations are missing
S4 failures exist
25.5 CLASS_E

Assign CLASS_E if:

the system violates physical constraints
claims heat destruction
claims impossible efficiency
claims self-powered data center from low-grade waste heat without valid balance
claims zero discharge without reject accounting
S5 failures exist
26. Machine-Readable Validation Template

The AI validator may use this output format:

sphere_dc_validation:
  package_version: "v0.1"
  verdict_class: "CLASS_B"
  primary_reason: ""
  module_status:
    thermal_core:
      status: "PASS"
      severity_max: "S0"
      notes: []
    w_ct:
      status: "WARN"
      severity_max: "S3"
      notes: []
    a_dc:
      status: "WARN"
      severity_max: "S3"
      notes: []
    w_env_dc:
      status: "WARN"
      severity_max: "S3"
      notes: []
    thermal_buffers:
      status: "PASS"
      severity_max: "S1"
      notes: []
    heat_export:
      status: "WARN"
      severity_max: "S3"
      notes: []
    auxiliary_power:
      status: "PASS"
      severity_max: "S1"
      notes: []
    integrated_balance:
      status: "WARN"
      severity_max: "S3"
      notes: []

  heat_balance:
    status: ""
    Q_total_kW: null
    Q_captured_kW: null
    Q_allocated_kW: null
    Q_tail_immediate_kW: null
    errors: []

  water_balance:
    status: ""
    V_makeup_original_m3_day: null
    V_clean_technical_m3_day: null
    V_final_liquid_reject_m3_day: null
    errors: []

  electric_balance:
    status: ""
    P_known_module_loads_kW: null
    P_aux_gross_kW: null
    P_external_required_known_kW: null
    missing_loads: []

  overclaim_audit:
    forbidden_claims_detected: []
    unresolved_claims: []

  sensitivity:
    top_variables: []

  site_data_gaps:
    P0: []
    P1: []
    P2: []
    P3: []

  final_notes:
    strongest_points: []
    weakest_points: []
    next_required_calculations: []
27. AI Prompt for External Evaluation

Use the following prompt when submitting the package to an AI evaluator:

Analyze the attached SPHERE-DC technical package as an engineering concept.

Do not evaluate it as a marketing text.

Check:
1. internal consistency,
2. heat balance,
3. water balance,
4. electric balance,
5. thermodynamic limits,
6. mass conservation,
7. double counting,
8. non-claim compliance,
9. missing site data,
10. dominant weak points.

Use the validation labels:
PASS, WARN, FAIL, UNKNOWN, NOT_APPLICABLE.

Use the severity labels:
S0, S1, S2, S3, S4, S5.

Classify the full package as:
CLASS_A, CLASS_B, CLASS_C, CLASS_D, or CLASS_E.

Do not assume zero heat, zero discharge, potable water, self-powered operation, or guaranteed economic viability unless explicitly proven.

Return a concise but technically strict validation report.
28. Expected Validation Outcome for Current v0.1 Reference Package

For the current v0.1 package, an honest AI evaluator should likely return:

Expected class: CLASS_B or CLASS_C

Reason:

The internal logic is mostly structured and physically constrained.
The package avoids core false claims.
However, feasibility depends strongly on site-specific data:
- actual heat capture architecture
- actual water chemistry
- actual air humidity profile
- actual external heat demand
- actual module electric loads
- actual reject handling

Expected strong points:

clear system boundary
explicit non-claim rules
module-level formulas
1 MW and 10 MW reference cases
mass and energy accounting
reject streams are preserved
auxiliary power is constrained by Carnot
heat export requires real demand
condensate requires humidity difference
thermal buffer is not treated as heat elimination

Expected weak points:

A-DC condensate assumptions may be aggressive
A-DC latent load may be large
W-CT recovery requires real chemistry data
W-env-DC recovery requires treatment validation
heat export requires a nearby real heat user
auxiliary power from low-grade heat is small
electric loads are incomplete
thermal tail remains significant
29. Final Validation Summary Formula Block
Q_total =
P_IT + P_facility_extra

Q_captured =
Q_total × eta_capture × eta_HX

Q_allocated =
Q_WCT
+ Q_ADC
+ Q_WENV
+ Q_export_candidate
+ Q_buffer_charge
+ Q_aux_used
+ Q_tail_captured_direct

Q_allocated <= Q_captured

Q_uncaptured_tail =
Q_total - Q_captured

Q_tail_immediate =
Q_uncaptured_tail
+ Q_tail_captured_direct
+ Q_aux_residual_to_tail
+ Q_export_loss_local
+ Q_buffer_overflow

V_WENV_in =
V_WCT_recovered
+ V_condensate_captured
+ V_external
+ V_rainwater
+ V_other_water

V_clean_technical =
V_WENV_treated_clean + V_WENV_bypass_clean

V_final_liquid_reject =
V_WCT_waste
+ V_WENV_reject
+ V_other_reject

fresh_water_reduction =
min(V_clean_technical, V_makeup_original)

P_aux_gross =
Q_aux_used × eta_aux_conversion × eta_power_electronics

P_external_SPHERE_required =
max(0, P_known_module_loads - P_aux_gross)

IF Q_allocated > Q_captured:
    FAIL S5

IF V_clean_technical > V_WENV_in:
    FAIL S5

IF eta_aux_conversion >= eta_Carnot_aux:
    FAIL S5

IF Q_export_useful > Q_export_demand:
    FAIL S5

IF self_powered_data_center_claim == true:
    FAIL S5

IF zero_discharge_claim == true AND V_final_liquid_reject > 0:
    FAIL S5

IF heat_elimination_claim == true:
    FAIL S5