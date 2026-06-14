# a-dc.md

# SPHERE-DC Technical Document 04: A-DC Module

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
A-DC = Air / Data Center Airflow Processing Module

Concept baseline:

SPHERE-DC intercepts selected air streams around or inside the data center boundary.
A-DC processes air for particulate reduction, humidity management, condensate recovery, and environmental load reduction.
A-DC must not compromise data center cooling, ventilation safety, or equipment operating conditions.
```

## 1. Scope

This document defines calculation logic for:

- air volume flow
- air mass flow
- humidity ratio
- dew point
- condensate potential
- particulate removal
- filter loading
- fan power
- air-side thermal load
- water recovery from condensation
- A-DC electric demand
- A-DC heat interaction with SPHERE thermal core
- failure and bypass conditions

This document does not define:

- final filter selection
- HVAC mechanical design
- duct geometry
- microbiological certification
- indoor air quality certification
- site permitting
- final control hardware

## 2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

A-DC priority condition:

Q_ADC is allocated after Q_WCT and before Q_WENV, Q_export, Q_buffer.

## 3. Required Inputs

### 3.1 Airflow Inputs

Symbol	Meaning	Unit
V_dot_air	Air volume flow through A-DC	m³/h
V_dot_air_m3_s	Air volume flow through A-DC	m³/s
rho_air	Air density	kg/m³
m_dot_air	Dry air mass flow	kg/s
P_atm	Atmospheric pressure	kPa
T_air_in	Inlet air temperature	°C
T_air_out	Outlet air temperature	°C
RH_in	Inlet relative humidity	0–1
RH_out	Outlet relative humidity	0–1
omega_in	Inlet humidity ratio	kg_water/kg_dry_air
omega_out	Outlet humidity ratio	kg_water/kg_dry_air
T_dew_in	Inlet dew point	°C
T_dew_out	Outlet dew point	°C

### 3.2 Filtration Inputs

Symbol	Meaning	Unit
PM_in	Inlet particulate concentration	µg/m³
PM_out	Outlet particulate concentration	µg/m³
eta_filter_PM	Particle removal efficiency	0–1
DeltaP_filter	Filter pressure drop	Pa
DeltaP_duct	Duct / module pressure drop	Pa
DeltaP_total_ADC	Total air-side pressure drop	Pa
DeltaP_other_ADC	Other A-DC pressure drop	Pa
M_PM_removed	Removed particulate mass	kg/day
M_filter_capacity	Filter dust holding capacity	kg
t_filter_life	Estimated filter service life	day

### 3.3 Fan / Electric Inputs

Symbol	Meaning	Unit
eta_fan	Fan efficiency	0–1
P_fan	Fan electric power	kW
P_controls_ADC	Sensors / control electric power	kW
P_pumps_ADC	Condensate pump electric power	kW
P_aux_ADC	Other A-DC electric loads	kW
P_ADC_electric	Total A-DC electric load	kW

### 3.4 Condensate / Water Inputs

Symbol	Meaning	Unit
m_dot_condensate	Condensate mass flow	kg/s
V_condensate	Condensate volume	m³/day
R_condensate_capture	Fraction of condensate captured and routed	0–1
V_condensate_captured	Captured condensate	m³/day
V_condensate_lost	Uncaptured condensate / drainage loss	m³/day
C_condensate_contaminant	Contaminant concentration in condensate	mg/L

### 3.5 Thermal Inputs

Symbol	Meaning	Unit
Cp_air	Specific heat capacity of air	kJ/kg·K
Q_sensible_ADC	Sensible air thermal load	kW
Q_latent_ADC	Latent thermal load from condensation	kW
Q_ADC_total	Total thermal interaction of A-DC	kW
Q_ADC_available	Heat available to A-DC from SPHERE thermal core	kW
Q_ADC_required	Heat required by A-DC process	kW
h_fg_water	Latent heat of water vaporization / condensation	kJ/kg

## 4. Default Constants

rho_air = 1.2 kg/m³
Cp_air = 1.005 kJ/kg·K
rho_water = 1000 kg/m³
h_fg_water_default = 2450 kJ/kg
P_atm_default = 101.325 kPa
seconds_per_day = 86400 s/day

Notes:

rho_air depends on temperature, pressure, and humidity.
Cp_air is approximate.
h_fg_water depends on temperature.
Psychrometric calculations should use local atmospheric pressure when available.

## 5. Air Mass Flow

V_dot_air_m3_s = V_dot_air / 3600
m_dot_air = rho_air × V_dot_air_m3_s

Equivalent:

m_dot_air = rho_air × V_dot_air / 3600

Constraint:

V_dot_air > 0
rho_air > 0
m_dot_air > 0

## 6. Humidity Ratio Model

### 6.1 Saturation Vapor Pressure

For first-order AI-evaluable calculation, use Magnus approximation:

p_ws(T) = 0.61094 × exp((17.625 × T) / (T + 243.04))

Where:

T in °C
p_ws in kPa

Valid approximation range:

0°C to 50°C typical engineering range

### 6.2 Water Vapor Partial Pressure

p_w = RH × p_ws(T)

Where:

RH is expressed as 0–1

### 6.3 Humidity Ratio

omega = 0.62198 × p_w / (P_atm - p_w)

Where:

omega in kg_water/kg_dry_air
P_atm in kPa
p_w in kPa

Constraints:

0 <= RH <= 1
p_w < P_atm
omega >= 0

## 7. Dew Point Model

Approximate dew point from temperature and relative humidity:

gamma = ln(RH) + (17.625 × T) / (243.04 + T)
T_dew = (243.04 × gamma) / (17.625 - gamma)

Where:

T in °C
RH in 0–1
T_dew in °C

Constraint:

RH > 0

## 8. Condensate Calculation

### 8.1 Condensation Condition

Condensation occurs only if:

omega_out < omega_in

If:

omega_out >= omega_in

Then:

m_dot_condensate = 0
V_condensate = 0

### 8.2 Condensate Mass Flow

m_dot_condensate = m_dot_air × max(0, omega_in - omega_out)

Where:

m_dot_condensate in kg/s

### 8.3 Condensate Volume Per Day

V_condensate = (m_dot_condensate / rho_water) × seconds_per_day

Where:

V_condensate in m³/day

### 8.4 Captured Condensate

V_condensate_captured = V_condensate × R_condensate_capture
V_condensate_lost = V_condensate - V_condensate_captured

Constraints:

0 <= R_condensate_capture <= 1
V_condensate_captured <= V_condensate
V_condensate_lost >= 0

## 9. Air-Side Thermal Load

### 9.1 Sensible Thermal Load

Q_sensible_ADC = m_dot_air × Cp_air × (T_air_in - T_air_out)

Where:

Q_sensible_ADC in kW
m_dot_air in kg/s
Cp_air in kJ/kg·K
temperature difference in K

Interpretation:

If Q_sensible_ADC > 0, air is cooled.
If Q_sensible_ADC < 0, air is heated.

### 9.2 Latent Thermal Load

Q_latent_ADC = m_dot_condensate × h_fg_water

Where:

Q_latent_ADC in kW
m_dot_condensate in kg/s
h_fg_water in kJ/kg

### 9.3 Total Air-Side Thermal Interaction

For cooling and dehumidification:

Q_ADC_total = Q_sensible_ADC + Q_latent_ADC

Constraint:

Do not count Q_ADC_total as useful heat reuse unless it is routed into a defined SPHERE process.

## 10. Filtration Calculation

### 10.1 Particle Removal Efficiency
eta_filter_PM = (PM_in - PM_out) / PM_in

Constraints:

PM_in > 0
PM_out >= 0
PM_out <= PM_in
0 <= eta_filter_PM <= 1

### 10.2 Removed Particulate Mass

M_PM_removed =
(PM_in - PM_out) × V_dot_air × 24 × 1e-9

Where:

PM concentration in µg/m³
V_dot_air in m³/h
M_PM_removed in kg/day
1 µg = 1e-9 kg
### 10.3 Filter Life Estimate

t_filter_life = M_filter_capacity / M_PM_removed

Constraint:

M_PM_removed > 0

Non-claim rule:

Filter lifetime must not be claimed without actual dust loading, humidity, particle size distribution, and manufacturer data.

## 11. Fan Power Calculation

### 11.1 Total Pressure Drop

DeltaP_total_ADC = DeltaP_filter + DeltaP_duct + DeltaP_other_ADC

### 11.2 Fan Power

P_fan = (DeltaP_total_ADC × V_dot_air_m3_s) / eta_fan / 1000

Where:

DeltaP_total_ADC in Pa
V_dot_air_m3_s in m³/s
P_fan in kW

Constraints:

DeltaP_total_ADC >= 0
0 < eta_fan <= 1

## 12. Total A-DC Electric Load

P_ADC_electric =
P_fan
+ P_controls_ADC
+ P_pumps_ADC
+ P_aux_ADC

Specific electric consumption per air volume:

SEC_air_ADC = P_ADC_electric / V_dot_air

Where:

SEC_air_ADC in kW per m³/h of airflow

Alternative daily electric intensity:

E_ADC_day = P_ADC_electric × 24
E_per_1000m3_air =
E_ADC_day / ((V_dot_air × 24) / 1000)

Where:

E_per_1000m3_air in kWh/1000 m³

## 13. Condensate Quality and Routing

A-DC condensate is not assumed clean by default.

Potential contaminants:

dust
metals
microorganisms
volatile compounds
filter residue
coil contamination
duct contamination
biological growth

Routing rule:

Captured condensate must be routed to W-env-DC or quality validation before reuse.

Direct reuse condition:

Direct reuse is allowed only if:
measured_condensate_quality <= required_reuse_quality_limits

Constraint:

V_condensate_captured must not be counted as clean technical water until treated or validated.

## 14. Humidity Safety Constraint

A-DC must avoid harmful over-drying or over-humidification.

Generic constraints:

RH_out >= RH_min_allowed
RH_out <= RH_max_allowed
T_dew_out <= dew_point_limit_for_target_zone

If serving data hall air:

A-DC output must remain inside data center environmental requirements.

If serving external environmental air:

A-DC output must not create local condensation, icing, corrosion, or biological growth risk.

No universal humidity target is assumed.

## 15. Heat Interaction With SPHERE Thermal Core

A-DC may require heat for:

filter regeneration
desiccant regeneration
anti-icing
coil drying
humidity control
condensate sanitation
air preheating

Heat demand variables:

Q_ADC_total = total air-side thermal load created or managed by the selected A-DC airflow, cooling, and dehumidification scenario.

Q_ADC_required = portion of A-DC process heat demand that can be supplied by the SPHERE thermal core.

Q_ADC_available = heat allocated to A-DC from the SPHERE thermal core.

Q_ADC_used = min(Q_ADC_available, Q_ADC_required)

Important distinction:

Q_ADC_total is not the same as Q_ADC in the integrated SPHERE-DC heat allocation.

Q_ADC_total represents total air-side sensible and latent thermal interaction.

Q_ADC, Q_ADC_available, and Q_ADC_used represent heat allocated from the captured SPHERE thermal core to A-DC heat-dependent processes.

Therefore:

Q_ADC_total may be greater than Q_ADC_available.

If Q_ADC_total exceeds Q_ADC_available, the difference must be handled by the existing HVAC system, auxiliary energy, reduced A-DC process load, reduced airflow, changed humidity target, or bypass.

Constraint:

Q_ADC_used <= Q_ADC_available

If:

Q_ADC_available < Q_ADC_required

Then one or more must occur:

reduce A-DC process load
reduce airflow through A-DC
use auxiliary heat
skip heat-dependent process
route air through bypass

## 16. Reference Case A: 1 MW Data Center

### 16.1 Declared Assumptions

P_IT = 1,000 kW

V_dot_air = 100,000 m³/h
rho_air = 1.2 kg/m³
Cp_air = 1.005 kJ/kg·K
P_atm = 101.325 kPa

T_air_in = 30°C
RH_in = 0.60

T_air_out = 20°C
RH_out = 0.50

DeltaP_filter = 400 Pa
DeltaP_duct = 100 Pa
DeltaP_other_ADC = 0 Pa
eta_fan = 0.65

PM_in = 25 µg/m³
PM_out = 5 µg/m³

R_condensate_capture = 0.90
h_fg_water = 2450 kJ/kg

### 16.2 Air Mass Flow

V_dot_air_m3_s = 100,000 / 3600
V_dot_air_m3_s = 27.78 m³/s
m_dot_air = 1.2 × 27.78
m_dot_air = 33.33 kg/s

### 16.3 Humidity Ratio

Using Magnus approximation:

omega_in at 30°C, RH 60% ≈ 0.0160 kg/kg
omega_out at 20°C, RH 50% ≈ 0.00725 kg/kg

Difference:

Delta_omega = 0.0160 - 0.00725
Delta_omega = 0.00875 kg/kg

### 16.4 Condensate Potential

m_dot_condensate = 33.33 × 0.00875
m_dot_condensate = 0.292 kg/s
V_condensate = (0.292 / 1000) × 86400
V_condensate = 25.2 m³/day

Captured condensate:

V_condensate_captured = 25.2 × 0.90
V_condensate_captured = 22.7 m³/day

Lost / uncaptured condensate:

V_condensate_lost = 25.2 - 22.7
V_condensate_lost = 2.5 m³/day

### 16.5 Thermal Load

Sensible:

Q_sensible_ADC = 33.33 × 1.005 × (30 - 20)
Q_sensible_ADC = 335 kW

Latent:

Q_latent_ADC = 0.292 × 2450
Q_latent_ADC = 715 kW

Total:

Q_ADC_total = 335 + 715
Q_ADC_total = 1050 kW

Interpretation:

This reference case requires more cooling/dehumidification load than the 1 MW IT load.
Therefore, this airflow and humidity target may be too aggressive unless the air stream and energy boundary are justified.

### 16.6 Fan Power

DeltaP_total_ADC = 400 + 100 + 0
DeltaP_total_ADC = 500 Pa
P_fan = (500 × 27.78) / 0.65 / 1000
P_fan = 21.4 kW

### 16.7 Particulate Removal

eta_filter_PM = (25 - 5) / 25
eta_filter_PM = 0.80
M_PM_removed =
(25 - 5) × 100,000 × 24 × 1e-9

M_PM_removed = 0.048 kg/day

## 17. Reference Case B: 10 MW Data Center

### 17.1 Declared Assumptions

Scale airflow linearly from Case A:

P_IT = 10,000 kW

V_dot_air = 1,000,000 m³/h
rho_air = 1.2 kg/m³
Cp_air = 1.005 kJ/kg·K
P_atm = 101.325 kPa

T_air_in = 30°C
RH_in = 0.60

T_air_out = 20°C
RH_out = 0.50

DeltaP_total_ADC = 500 Pa
eta_fan = 0.65

PM_in = 25 µg/m³
PM_out = 5 µg/m³

R_condensate_capture = 0.90
h_fg_water = 2450 kJ/kg

### 17.2 Air Mass Flow

V_dot_air_m3_s = 1,000,000 / 3600
V_dot_air_m3_s = 277.78 m³/s
m_dot_air = 1.2 × 277.78
m_dot_air = 333.33 kg/s
### 17.3 Humidity Ratio

omega_in at 30°C, RH 60% ≈ 0.0160 kg/kg
omega_out at 20°C, RH 50% ≈ 0.00725 kg/kg
Delta_omega = 0.00875 kg/kg

### 17.4 Condensate Potential

m_dot_condensate = 333.33 × 0.00875
m_dot_condensate = 2.92 kg/s
V_condensate = (2.92 / 1000) × 86400
V_condensate = 252 m³/day

Captured condensate:

V_condensate_captured = 252 × 0.90
V_condensate_captured = 227 m³/day

Lost / uncaptured condensate:

V_condensate_lost = 252 - 227
V_condensate_lost = 25 m³/day

### 17.5 Thermal Load

Sensible:

Q_sensible_ADC = 333.33 × 1.005 × (30 - 20)
Q_sensible_ADC = 3350 kW

Latent:

Q_latent_ADC = 2.92 × 2450
Q_latent_ADC = 7150 kW

Total:

Q_ADC_total = 3350 + 7150
Q_ADC_total = 10,500 kW

Interpretation:

This reference case requires ~10.5 MW of cooling/dehumidification load.
For a 10 MW IT data center, the assumed airflow and humidity target represent a major energy boundary and must be site-validated.

### 17.6 Fan Power

P_fan = (500 × 277.78) / 0.65 / 1000
P_fan = 213.7 kW

### 17.7 Particulate Removal

eta_filter_PM = (25 - 5) / 25
eta_filter_PM = 0.80
M_PM_removed =
(25 - 5) × 1,000,000 × 24 × 1e-9

M_PM_removed = 0.48 kg/day

## 18. Interpretation Rules
A-DC condensate is climate-dependent.
A-DC condensate is humidity-dependent.
A-DC condensate is airflow-dependent.
A-DC condensate must not be counted as clean water before validation or treatment.
A-DC filtration benefit must be balanced against fan power and filter replacement.
A-DC dehumidification can create large latent thermal load.
A-DC must not be modeled as free water generation.
A-DC must not violate data center environmental constraints.

## 19. Process Output Requirements

A-DC calculation must output:

V_dot_air
m_dot_air
T_air_in
RH_in
omega_in
T_dew_in
T_air_out
RH_out
omega_out
T_dew_out
m_dot_condensate
V_condensate
V_condensate_captured
V_condensate_lost
Q_sensible_ADC
Q_latent_ADC
Q_ADC_total
PM_in
PM_out
eta_filter_PM
M_PM_removed
DeltaP_total_ADC
P_fan
P_ADC_electric
Q_ADC_required
Q_ADC_available
Q_ADC_used

## 20. Control Logic

INPUT:
    V_dot_air
    T_air_in
    RH_in
    PM_in
    target_T_air_out
    target_RH_out
    filter_status
    Q_ADC_available
    condensate_quality

PROCESS:
    calculate m_dot_air
    calculate omega_in
    calculate omega_out
    calculate condensate potential
    calculate Q_sensible_ADC
    calculate Q_latent_ADC
    calculate Q_ADC_total
    calculate filter removal
    calculate fan power
    check humidity safety limits
    check recovered condensate quality
    compare Q_ADC_required with Q_ADC_available

IF humidity_outside_allowed_range:
    adjust airflow, temperature target, or bypass

IF filter_pressure_drop_too_high:
    reduce airflow or replace filter

IF condensate_quality_valid:
    route condensate to W-env-DC or reuse path

ELSE:
    route condensate to W-env-DC treatment or reject

IF Q_ADC_available < Q_ADC_required:
    reduce heat-dependent process load
    use auxiliary energy
    or bypass heat-dependent process

OUTPUT:
    processed air
    captured condensate
    particulate waste
    updated electric load
    updated thermal interaction

## 21. Failure / Bypass Conditions

IF ADC_fault == true:
    isolate A-DC
    preserve primary data center airflow / HVAC safety
    route air through bypass
    route Q_ADC allocation to next SPHERE priority module
IF DeltaP_total_ADC > DeltaP_max_allowed:
    reduce airflow
    trigger filter maintenance
    or bypass A-DC
IF RH_out < RH_min_allowed:
    reduce dehumidification
    bypass dehumidification stage
    or rehumidify if required
IF RH_out > RH_max_allowed:
    increase dehumidification if energy available
    or bypass reuse claim
IF condensate_quality_out_of_spec:
    block direct reuse
    route to W-env-DC or reject
IF biological_growth_risk == true:
    stop direct reuse
    sanitize condensate path
    inspect surfaces

## 22. Engineering Constraints

V_dot_air > 0

m_dot_air > 0

0 <= RH_in <= 1

0 <= RH_out <= 1

omega_in >= 0

omega_out >= 0

m_dot_condensate >= 0

V_condensate_captured <= V_condensate

PM_out <= PM_in

0 <= eta_filter_PM <= 1

DeltaP_total_ADC <= DeltaP_max_allowed

0 < eta_fan <= 1

P_ADC_electric must be included in system electric load.

A-DC must not reduce primary cooling reliability.

A-DC must be bypassable.

Condensate must not be counted as clean technical water before validation or W-env-DC treatment.

## 23. AI Validation Questions

An AI analysis system should verify:

1. Are air volume flow and air mass flow consistent?

2. Are RH values expressed consistently as 0–1 or %?

3. Is atmospheric pressure included in humidity ratio calculation?

4. Is omega_out lower than omega_in before condensate is claimed?

5. Is condensate volume climate-dependent and not assumed constant?

6. Is latent load calculated when condensation is claimed?

7. Does Q_ADC_total exceed reasonable available energy?

8. Is fan power calculated from pressure drop and airflow?

9. Is filter pressure drop included in electric load?

10. Is particulate mass removal calculated from concentration and airflow?

11. Is filter waste tracked?

12. Is condensate quality validated before reuse?

13. Is over-drying avoided?

14. Is biological growth risk considered?

15. Is A-DC bypass defined?

16. Is A-DC preserving primary data center HVAC safety?

17. Does the model avoid claiming free water generation?

18. Does the model avoid counting condensate twice in W-env-DC?

19. Are 1 MW and 10 MW cases scaled from declared assumptions?

20. Which assumptions dominate feasibility: airflow, RH_in, RH_out, DeltaP_total_ADC, or Q_ADC_total?

## 24. Minimum Site Data Required
airflow source definition
airflow rate
air temperature profile
relative humidity profile
local atmospheric pressure
seasonal climate data
data hall environmental limits
external air quality data
PM2.5 concentration
PM10 concentration
VOC data if relevant
filter class
filter pressure drop curve
dust holding capacity
fan efficiency
duct pressure drop
allowed pressure drop
condensate quality
condensate biological risk
cooling coil or dehumidification architecture
existing HVAC/BMS integration points
allowed humidity range
allowed temperature range
water reuse quality target
available Q_ADC from SPHERE thermal core
A-DC maintenance access constraints

## 25. Non-Claim Rules

Not allowed without site-specific proof:

free water generation
constant condensate production
clean condensate without treatment
zero air pollution
complete particle removal
universal humidity benefit
no fan energy penalty
no filter maintenance penalty
safe direct reuse of condensate
guaranteed local climate improvement

Allowed claim format:

Under declared assumptions:
A-DC processes X m³/h of air.
A-DC may condense Y m³/day of water if omega_in > omega_out.
A-DC captures Z m³/day of condensate at R_condensate_capture.
A-DC removes M kg/day of particulate matter under declared PM assumptions.
A-DC requires P kW of fan power under declared pressure drop.
A-DC condensate requires validation or W-env-DC treatment before reuse.

## 26. Summary Formula Block

V_dot_air_m3_s = V_dot_air / 3600

m_dot_air = rho_air × V_dot_air_m3_s

p_ws(T) = 0.61094 × exp((17.625 × T) / (T + 243.04))

p_w = RH × p_ws(T)

omega = 0.62198 × p_w / (P_atm - p_w)

gamma = ln(RH) + (17.625 × T) / (243.04 + T)

T_dew = (243.04 × gamma) / (17.625 - gamma)

m_dot_condensate = m_dot_air × max(0, omega_in - omega_out)

V_condensate = (m_dot_condensate / rho_water) × seconds_per_day

V_condensate_captured = V_condensate × R_condensate_capture

V_condensate_lost = V_condensate - V_condensate_captured

Q_sensible_ADC = m_dot_air × Cp_air × (T_air_in - T_air_out)

Q_latent_ADC = m_dot_condensate × h_fg_water

Q_ADC_total = Q_sensible_ADC + Q_latent_ADC

eta_filter_PM = (PM_in - PM_out) / PM_in

M_PM_removed =
(PM_in - PM_out) × V_dot_air × 24 × 1e-9

DeltaP_total_ADC = DeltaP_filter + DeltaP_duct + DeltaP_other_ADC

P_fan = (DeltaP_total_ADC × V_dot_air_m3_s) / eta_fan / 1000

P_ADC_electric =
P_fan
+ P_controls_ADC
+ P_pumps_ADC
+ P_aux_ADC

Q_ADC_used = min(Q_ADC_available, Q_ADC_required)
