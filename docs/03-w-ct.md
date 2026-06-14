# w-ct.md

# SPHERE-DC Technical Document 03: W-CT Module

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
W-CT = Water / Cooling Tower Blowdown Treatment Module

Concept baseline:

SPHERE-DC intercepts cooling tower blowdown before discharge.
W-CT receives concentrated cooling tower water containing salts, biocides, inhibitors, and metals.
W-CT uses available data center heat and auxiliary equipment to recover technical water and concentrate the remaining contaminants into a smaller waste stream.

W-CT is the SPHERE-DC module for cooling tower blowdown treatment, with blowdown capture before external discharge, technical water recovery, and concentration of salts, metals, and treatment chemicals into brine, sludge, or reject.
```

## 1. Scope

This document defines calculation logic for:

cooling tower evaporation
cooling tower blowdown
make-up water demand
blowdown contaminant load
recoverable water volume
remaining brine / sludge / reject volume
heat demand of W-CT
electric demand of W-CT
water balance impact
discharge reduction

This document does not define:

final equipment selection
membrane type
evaporator type
chemical dosing design
hazardous waste classification
site permitting
final water quality certification

## 2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

W-CT priority condition:

Q_WCT is allocated before Q_ADC, Q_WENV, Q_export, Q_buffer.

## 3. Required Inputs

### 3.1 Data Center / Cooling Tower Inputs
Symbol	Meaning	Unit
P_IT	IT electrical load	kW
Q_total	Total data center heat load inside selected boundary	kW
Q_CT	Heat load rejected through cooling tower boundary	kW
V_circ_CT	Cooling tower circulating water flow	m³/day
CoC	Cycles of concentration	dimensionless
V_makeup_original	Original cooling tower make-up water demand	m³/day
V_blowdown_original	Original cooling tower blowdown volume	m³/day
V_evap	Evaporative loss	m³/day
V_drift	Drift loss	m³/day
V_leak	Leakage / miscellaneous loss	m³/day

### 3.2 Water Chemistry Inputs

Symbol	Meaning	Unit
TDS_makeup	TDS in make-up water	mg/L
TDS_blowdown	TDS in blowdown	mg/L
EC_blowdown	Electrical conductivity of blowdown	µS/cm
pH_blowdown	pH of blowdown	pH
TSS_blowdown	Total suspended solids	mg/L
hardness_blowdown	Hardness	mg/L as CaCO₃
chloride_blowdown	Chloride concentration	mg/L
silica_blowdown	Silica concentration	mg/L
metals_blowdown	Sum or selected metals concentration	mg/L
biocide_residual	Biocide residual concentration	mg/L
inhibitor_residual	Corrosion / scale inhibitor concentration	mg/L

### 3.3 W-CT Process Inputs

Symbol	Meaning	Unit
R_WCT	W-CT water recovery rate	0–1
V_WCT_in	W-CT inlet water volume	m³/day
V_WCT_recovered	Recovered technical water from W-CT	m³/day
V_WCT_waste	Concentrated brine / sludge / reject volume	m³/day
CF_WCT	Concentration factor of W-CT waste stream	dimensionless
Q_WCT_available	Heat available to W-CT	kW
Q_WCT_required	Heat required by W-CT process	kW
P_WCT_electric	W-CT electric load	kW
eta_WCT_thermal_use	Fraction of W-CT process heat supplied by captured DC heat	0–1

## 4. Default Constants

rho_water = 1000 kg/m³
h_fg_water_evap_default = 2450 kJ/kg
seconds_per_day = 86400 s/day
liters_per_m3 = 1000 L/m³

## 5. Core Cooling Tower Balance

### 5.1 First-Order Evaporation From Heat Rejection

m_evap_kg_s = Q_CT / h_fg_water_evap_default
V_evap = (m_evap_kg_s / rho_water) × seconds_per_day

Where:

Q_CT in kW = kJ/s
h_fg_water_evap_default in kJ/kg
m_evap_kg_s in kg/s
V_evap in m³/day

### 5.2 Blowdown From Cycles of Concentration

V_blowdown = V_evap / (CoC - 1)

Constraint:

CoC > 1
### 5.3 Cooling Tower Make-Up Water

V_makeup = V_evap + V_blowdown + V_drift + V_leak

If V_drift and V_leak are unknown:

V_makeup ≈ V_evap + V_blowdown

## 6. W-CT Inlet Definition

Base case:

V_WCT_in = V_blowdown

Extended case:

V_WCT_in = V_blowdown + V_other_technical_waste

Constraint:

V_WCT_in >= V_blowdown

## 7. Water Recovery Calculation

### 7.1 Recovered Technical Water

V_WCT_recovered = V_WCT_in × R_WCT

### 7.2 Remaining Waste Stream

V_WCT_waste = V_WCT_in - V_WCT_recovered

Equivalent:

V_WCT_waste = V_WCT_in × (1 - R_WCT)

Constraints:

0 <= R_WCT < 1

V_WCT_recovered <= V_WCT_in

V_WCT_waste > 0

Non-claim rule:

R_WCT = 1.0 is not allowed in concept claims.
100% blowdown recovery is not allowed.

## 8. Contaminant Mass Balance

For any dissolved or suspended component X:

M_X_in_kg_day = V_WCT_in × 1000 × C_X_in / 1,000,000

Where:

V_WCT_in in m³/day
1000 = liters per m³
C_X_in in mg/L
M_X_in_kg_day in kg/day

Simplified:

M_X_in_kg_day = V_WCT_in × C_X_in / 1000

Because:

m³/day × mg/L / 1000 = kg/day

### 8.1 Recovered Water Contaminant Load

M_X_recovered_kg_day = V_WCT_recovered × C_X_recovered / 1000
8.2 Waste Stream Contaminant Load
M_X_waste_kg_day = M_X_in_kg_day - M_X_recovered_kg_day

Constraint:

M_X_waste_kg_day >= 0
8.3 Waste Stream Concentration
C_X_waste = (M_X_waste_kg_day × 1000) / V_WCT_waste

Where:

C_X_waste in mg/L

## 9. TDS Approximation

If only make-up water TDS and CoC are known:

TDS_blowdown ≈ TDS_makeup × CoC

TDS mass entering W-CT:

M_TDS_in_kg_day = V_WCT_in × TDS_blowdown / 1000

If recovered water target TDS is known:

M_TDS_recovered_kg_day = V_WCT_recovered × TDS_recovered / 1000

TDS remaining in waste:

M_TDS_waste_kg_day = M_TDS_in_kg_day - M_TDS_recovered_kg_day

Waste TDS:

TDS_waste = (M_TDS_waste_kg_day × 1000) / V_WCT_waste

## 10. W-CT Concentration Factor

Volume-based concentration factor:

CF_WCT_volume = V_WCT_in / V_WCT_waste

If recovery rate is used:

CF_WCT_volume = 1 / (1 - R_WCT)

Examples:

R_WCT = 0.50 → CF_WCT_volume = 2
R_WCT = 0.75 → CF_WCT_volume = 4
R_WCT = 0.90 → CF_WCT_volume = 10

Constraint:

Higher CF_WCT increases waste concentration and scaling risk.

## 11. Heat Demand Model

W-CT may use heat for:

preheating
evaporation
thermal concentration
membrane support
regeneration
sludge drying

### 11.1 Thermal Evaporation Requirement

If W-CT uses thermal evaporation to recover water:

Q_WCT_required_kW = (m_evap_WCT_kg_s × h_fg_effective) / eta_WCT_thermal_process

Where:

m_evap_WCT_kg_s = water evaporated inside W-CT, kg/s
h_fg_effective = effective latent heat requirement, kJ/kg
eta_WCT_thermal_process = effective thermal process efficiency, 0–1

If recovered water is assumed to come from evaporation / condensation:

m_evap_WCT_kg_s = (V_WCT_recovered × rho_water) / seconds_per_day

Then:

Q_WCT_required_kW =
((V_WCT_recovered × rho_water) / seconds_per_day × h_fg_effective)
/
eta_WCT_thermal_process

### 11.2 Available Heat Constraint

Q_WCT_used = min(Q_WCT_available, Q_WCT_required)

If:

Q_WCT_available < Q_WCT_required

Then one or more must occur:

reduce R_WCT
increase residence time
use auxiliary heat
switch to lower-energy process
route remaining water to conventional treatment

## 12. Electric Load Model

W-CT electric load:

P_WCT_electric =
P_pumps_WCT
+ P_controls_WCT
+ P_filtration_WCT
+ P_membrane_WCT
+ P_sludge_handling_WCT

Specific electric consumption:

SEC_WCT = P_WCT_electric × 24 / V_WCT_recovered

Where:

SEC_WCT in kWh/m³
P_WCT_electric in kW
V_WCT_recovered in m³/day

Constraint:

V_WCT_recovered > 0

## 13. Cooling Tower Water Impact

### 13.1 New Make-Up Water Demand

If recovered W-CT water is reused as cooling tower make-up:

V_makeup_new = V_makeup_original - V_WCT_reused_as_makeup

Constraint:

0 <= V_WCT_reused_as_makeup <= V_WCT_recovered
V_makeup_new >= 0

### 13.2 Fresh Water Reduction

fresh_water_reduction_WCT = V_makeup_original - V_makeup_new

Equivalent:

fresh_water_reduction_WCT = min(V_WCT_recovered, V_makeup_original)

### 13.3 Liquid Discharge Reduction

Original discharge:

V_discharge_original = V_blowdown_original

New discharge:

V_discharge_new = V_WCT_waste + V_unprocessed_blowdown

Discharge reduction:

liquid_discharge_reduction_WCT =
V_discharge_original - V_discharge_new

Discharge reduction fraction:

discharge_reduction_fraction_WCT =
liquid_discharge_reduction_WCT / V_discharge_original

Constraints:

V_discharge_new >= 0
0 <= discharge_reduction_fraction_WCT <= 1

## 14. Reference Case A: 1 MW Data Center

### 14.1 Given / Assumed Inputs

From SPHERE-DC baseline:

P_IT = 1,000 kW
V_circ_CT ≈ 4,500–5,000 m³/day
total water loss ≈ 70 m³/day

For this reference calculation:

Q_CT = 1,000 kW
CoC = 3
V_drift = 0
V_leak = 0
R_WCT = 0.75
TDS_makeup = 500 mg/L
TDS_blowdown ≈ TDS_makeup × CoC
TDS_recovered = 250 mg/L

### 14.2 Evaporation Estimate

m_evap_kg_s = 1000 / 2450
m_evap_kg_s = 0.408 kg/s
V_evap = (0.408 / 1000) × 86400
V_evap = 35.3 m³/day

### 14.3 Blowdown Estimate

V_blowdown = 35.3 / (3 - 1)
V_blowdown = 17.65 m³/day

### 14.4 Make-Up Water Estimate

V_makeup ≈ V_evap + V_blowdown
V_makeup ≈ 35.3 + 17.65
V_makeup ≈ 52.95 m³/day

Note:

The SPHERE-DC reference baseline gives ~70 m³/day total water loss for 1 MW.
The difference may be explained by drift, leakage, site design, climate, tower efficiency, or broader boundary assumptions.

### 14.5 W-CT Recovery

V_WCT_in = V_blowdown
V_WCT_in = 17.65 m³/day
V_WCT_recovered = 17.65 × 0.75
V_WCT_recovered = 13.24 m³/day
V_WCT_waste = 17.65 - 13.24
V_WCT_waste = 4.41 m³/day
CF_WCT_volume = 17.65 / 4.41
CF_WCT_volume ≈ 4.0

### 14.6 TDS Mass Balance

TDS_blowdown = 500 × 3
TDS_blowdown = 1500 mg/L
M_TDS_in = 17.65 × 1500 / 1000
M_TDS_in = 26.48 kg/day
M_TDS_recovered = 13.24 × 250 / 1000
M_TDS_recovered = 3.31 kg/day
M_TDS_waste = 26.48 - 3.31
M_TDS_waste = 23.17 kg/day
TDS_waste = (23.17 × 1000) / 4.41
TDS_waste ≈ 5254 mg/L

### 14.7 Discharge Reduction

Assume all blowdown enters W-CT:

V_discharge_original = 17.65 m³/day
V_discharge_new = 4.41 m³/day
liquid_discharge_reduction_WCT = 17.65 - 4.41
liquid_discharge_reduction_WCT = 13.24 m³/day
discharge_reduction_fraction_WCT = 13.24 / 17.65
discharge_reduction_fraction_WCT = 0.75

Result:

W-CT reduces liquid blowdown discharge by 75% under declared assumptions.
Remaining concentrated waste = 4.41 m³/day.

## 15. Reference Case B: 10 MW Data Center

### 15.1 Given / Scaled Inputs

Scale from 1 MW reference:

P_IT = 10,000 kW
Q_CT = 10,000 kW
V_circ_CT ≈ 45,000–50,000 m³/day
total water loss ≈ 700 m³/day

For this calculation:

CoC = 3
V_drift = 0
V_leak = 0
R_WCT = 0.75
TDS_makeup = 500 mg/L
TDS_blowdown ≈ TDS_makeup × CoC
TDS_recovered = 250 mg/L

### 15.2 Evaporation Estimate

m_evap_kg_s = 10000 / 2450
m_evap_kg_s = 4.082 kg/s
V_evap = (4.082 / 1000) × 86400
V_evap = 352.8 m³/day

### 15.3 Blowdown Estimate

V_blowdown = 352.8 / (3 - 1)
V_blowdown = 176.4 m³/day

### 15.4 Make-Up Water Estimate

V_makeup ≈ 352.8 + 176.4
V_makeup ≈ 529.2 m³/day

Note:

The scaled SPHERE-DC baseline suggests ~700 m³/day total water loss for 10 MW.
The difference depends on drift, leakage, climate, cooling tower design, CoC, and boundary definition.

### 15.5 W-CT Recovery

V_WCT_in = 176.4 m³/day
V_WCT_recovered = 176.4 × 0.75
V_WCT_recovered = 132.3 m³/day
V_WCT_waste = 176.4 - 132.3
V_WCT_waste = 44.1 m³/day
CF_WCT_volume = 176.4 / 44.1
CF_WCT_volume = 4.0

### 15.6 TDS Mass Balance

TDS_blowdown = 500 × 3
TDS_blowdown = 1500 mg/L
M_TDS_in = 176.4 × 1500 / 1000
M_TDS_in = 264.6 kg/day
M_TDS_recovered = 132.3 × 250 / 1000
M_TDS_recovered = 33.08 kg/day
M_TDS_waste = 264.6 - 33.08
M_TDS_waste = 231.52 kg/day
TDS_waste = (231.52 × 1000) / 44.1
TDS_waste ≈ 5250 mg/L

### 15.7 Discharge Reduction

V_discharge_original = 176.4 m³/day
V_discharge_new = 44.1 m³/day
liquid_discharge_reduction_WCT = 176.4 - 44.1
liquid_discharge_reduction_WCT = 132.3 m³/day
discharge_reduction_fraction_WCT = 132.3 / 176.4
discharge_reduction_fraction_WCT = 0.75

Result:

W-CT reduces liquid blowdown discharge by 75% under declared assumptions.
Remaining concentrated waste = 44.1 m³/day.

## 16. Heat Requirement Sensitivity

If recovered water is produced by thermal evaporation:

Q_WCT_required_kW =
((V_WCT_recovered × rho_water) / seconds_per_day × h_fg_effective)
/
eta_WCT_thermal_process

Example for 1 MW case:

V_WCT_recovered = 13.24 m³/day
h_fg_effective = 2450 kJ/kg
eta_WCT_thermal_process = 1.0
m_evap_WCT = (13.24 × 1000) / 86400
m_evap_WCT = 0.153 kg/s
Q_WCT_required = 0.153 × 2450
Q_WCT_required = 375 kW

Example for 10 MW case:

V_WCT_recovered = 132.3 m³/day
m_evap_WCT = (132.3 × 1000) / 86400
m_evap_WCT = 1.531 kg/s
Q_WCT_required = 1.531 × 2450
Q_WCT_required = 3751 kW

Interpretation rule:

If W-CT is modeled as pure thermal evaporation, heat demand is high.
Lower-energy processes or multi-effect / heat recovery designs must be modeled explicitly.

Non-claim rule:

Do not claim high W-CT recovery with low heat demand unless process architecture supports it.

## 17. Process Output Requirements

W-CT calculation must output:

V_evap
V_blowdown
V_makeup
V_WCT_in
V_WCT_recovered
V_WCT_waste
CF_WCT_volume
M_TDS_in
M_TDS_recovered
M_TDS_waste
TDS_waste
Q_WCT_required
Q_WCT_available
Q_WCT_used
P_WCT_electric
SEC_WCT
fresh_water_reduction_WCT
liquid_discharge_reduction_WCT
discharge_reduction_fraction_WCT

## 18. Control Logic

INPUT:
    V_blowdown
    water_chemistry
    Q_WCT_available
    R_WCT_target
    recovered_water_quality_target

PROCESS:
    measure V_blowdown
    measure TDS, EC, pH, TSS, ORP, biocide_residual
    calculate V_WCT_in
    calculate maximum feasible R_WCT
    calculate Q_WCT_required
    compare Q_WCT_required with Q_WCT_available
    calculate V_WCT_recovered
    calculate V_WCT_waste
    calculate waste concentration
    verify recovered water quality

IF recovered_water_quality <= target:
    route recovered water to cooling tower make-up or W-env-DC

ELSE:
    route recovered water to W-env-DC polishing
    or reduce R_WCT
    or route to conventional treatment

IF Q_WCT_available < Q_WCT_required:
    reduce recovery rate
    increase process time
    use auxiliary energy
    or route excess blowdown to fallback treatment

OUTPUT:
    recovered technical water
    concentrated brine/sludge/reject
    updated make-up water demand
    updated liquid discharge

## 19. Failure / Bypass Conditions

IF WCT_fault == true:
    isolate W-CT
    preserve cooling tower operation
    route blowdown to existing treatment/discharge path
    route Q_WCT allocation to next SPHERE priority module
IF recovered_water_quality_out_of_spec == true:
    block reuse as cooling tower make-up
    route to W-env-DC or fallback treatment
IF waste_tank_full == true:
    stop recovery process
    preserve blowdown safety route
IF pH_out_of_range == true:
    stop reuse
    require neutralization or fallback treatment
IF conductivity_high == true:
    stop direct reuse
    route to W-env-DC polishing or reject stream

## 20. Engineering Constraints

CoC > 1

0 <= R_WCT < 1

V_WCT_recovered <= V_WCT_in

V_WCT_waste > 0

Q_WCT_used <= Q_WCT_available

Recovered water quality must satisfy cooling tower make-up requirements before reuse.

Concentrated waste must be handled as industrial waste unless site-specific analysis proves otherwise.

W-CT must not interrupt primary cooling tower safety operation.

W-CT must be bypassable.

No zero-discharge claim is allowed without full site-specific mass balance and regulatory validation.

## 21. AI Validation Questions

An AI analysis system should verify:

1. Are V_evap, V_blowdown, and V_makeup internally consistent?

2. Is CoC > 1?

3. Is the assumed CoC realistic for the declared water chemistry?

4. Is V_blowdown calculated from evaporation and CoC, or imported from real site data?

5. If real site data exists, does calculated V_blowdown match measured V_blowdown?

6. Is R_WCT declared as an assumption?

7. Is V_WCT_recovered <= V_WCT_in?

8. Is V_WCT_waste > 0?

9. Are salts/metals/biocides/inhibitors tracked into the waste stream?

10. Is contaminant mass conserved?

11. Is recovered water quality specified before reuse?

12. Is W-CT heat demand calculated if thermal recovery is claimed?

13. Does Q_WCT_required exceed available captured heat?

14. Is electric load subtracted from system benefit?

15. Is remaining waste volume explicitly reported?

16. Is discharge reduction calculated from volume only, or also contaminant mass?

17. Is zero-discharge avoided as a claim?

18. Is fallback routing defined if W-CT fails?

19. Is waste handling included as an output, not hidden?

20. Which variables dominate feasibility: CoC, R_WCT, TDS, Q_WCT_required, or disposal cost?

## 22. Minimum Site Data Required

cooling tower type
cooling tower heat rejection load
cooling tower circulating water flow
cooling tower make-up water volume
cooling tower blowdown volume
cycles of concentration
evaporation estimate or measured evaporation
drift loss
leakage / miscellaneous loss

make-up water TDS
blowdown TDS
conductivity
pH
TSS
hardness
chloride
silica
metals
biocide residual
corrosion inhibitor residual
scaling index
microbiological constraints

allowed recovered water quality for reuse
existing blowdown treatment method
existing discharge cost
industrial waste disposal cost
available heat to W-CT
available electrical power to W-CT
permitted waste storage volume
BMS / SCADA integration points

## 23. Non-Claim Rules

Not allowed without site-specific proof:

100% water recovery
zero blowdown
zero liquid discharge
complete elimination of wastewater
safe direct discharge of concentrated waste
universal economic viability
guaranteed payback
recovered water suitable for reuse without quality testing

Allowed claim format:

Under declared assumptions:
W-CT reduces liquid blowdown discharge from X m³/day to Y m³/day.
W-CT recovers Z m³/day of technical water.
W-CT concentrates remaining contaminants into W m³/day of waste stream.
Recovered water requires quality validation before reuse.

## 24. Summary Formula Block

m_evap_kg_s = Q_CT / h_fg_water_evap_default

V_evap = (m_evap_kg_s / rho_water) × seconds_per_day

V_blowdown = V_evap / (CoC - 1)

V_makeup = V_evap + V_blowdown + V_drift + V_leak

V_WCT_in = V_blowdown + V_other_technical_waste

V_WCT_recovered = V_WCT_in × R_WCT

V_WCT_waste = V_WCT_in - V_WCT_recovered

CF_WCT_volume = V_WCT_in / V_WCT_waste

TDS_blowdown ≈ TDS_makeup × CoC

M_X_in_kg_day = V_WCT_in × C_X_in / 1000

M_X_recovered_kg_day = V_WCT_recovered × C_X_recovered / 1000

M_X_waste_kg_day = M_X_in_kg_day - M_X_recovered_kg_day

C_X_waste = (M_X_waste_kg_day × 1000) / V_WCT_waste

Q_WCT_required_kW =
((V_WCT_recovered × rho_water) / seconds_per_day × h_fg_effective)
/
eta_WCT_thermal_process

Q_WCT_used = min(Q_WCT_available, Q_WCT_required)

P_WCT_electric =
P_pumps_WCT
+ P_controls_WCT
+ P_filtration_WCT
+ P_membrane_WCT
+ P_sludge_handling_WCT

SEC_WCT = P_WCT_electric × 24 / V_WCT_recovered

V_makeup_new = V_makeup_original - V_WCT_reused_as_makeup

fresh_water_reduction_WCT = min(V_WCT_reused_as_makeup, V_makeup_original)

V_discharge_new = V_WCT_waste + V_unprocessed_blowdown

liquid_discharge_reduction_WCT =
V_discharge_original - V_discharge_new

discharge_reduction_fraction_WCT =
liquid_discharge_reduction_WCT / V_discharge_original
