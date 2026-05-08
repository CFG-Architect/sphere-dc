# thermal-buffers.md

# SPHERE-DC Technical Document 06: Thermal Buffers

## 0. Document Type

AI-evaluable engineering specification.

Module:

```text
Thermal Buffers = heat storage and release layer inside SPHERE-DC

Concept baseline:

SPHERE-DC uses thermal buffers to absorb temporary excess heat, smooth module demand, reduce short-duration thermal tail spikes, and provide time-shifted heat to W-CT, A-DC, W-env-DC, heat export, or auxiliary processes.

Thermal buffers do not eliminate heat.
Thermal buffers only delay, redistribute, or stabilize heat flows.
```

1. Scope

This document defines calculation logic for:

thermal peak absorption
required buffer energy capacity
charge power
discharge power
water-based thermal storage
phase-change material storage
solid media storage
thermal losses
usable stored energy
buffer state of charge
buffer full condition
buffer empty condition
thermal tail interaction
failure and bypass conditions

This document does not define:

final tank design
final insulation design
pressure vessel certification
material selection
phase-change material selection
fire safety certification
structural integration
detailed control hardware
2. Module Position in SPHERE-DC Priority

Heat routing priority:

1. W-CT
2. A-DC
3. W-env-DC
4. Heat export
5. Thermal buffers
6. Thermal tail

Thermal buffer priority condition:

Q_buffer is allocated after Q_WCT, Q_ADC, Q_WENV, and Q_export.

If buffer capacity is available:
    route remaining captured heat to buffer.

If buffer is full:
    route remaining heat to thermal tail.
3. Required Inputs
3.1 Heat Flow Inputs
Symbol	Meaning	Unit
Q_total	Total heat load inside selected boundary	kW
Q_captured	Heat captured by SPHERE thermal core	kW
Q_WCT	Heat routed to W-CT	kW
Q_ADC	Heat routed to A-DC	kW
Q_WENV	Heat routed to W-env-DC	kW
Q_export	Heat routed to heat export	kW
Q_buffer_in_available	Heat available for buffer charging	kW
Q_buffer_charge	Actual buffer charge power	kW
Q_buffer_discharge	Actual buffer discharge power	kW
Q_tail_total	Total residual heat after module allocation	kW
3.2 Peak Inputs
Symbol	Meaning	Unit
Q_peak_excess	Temporary excess heat above useful module demand	kW
t_peak	Peak duration	h
N_peak_events	Number of peak events per day	count/day
safety_factor_buffer	Buffer sizing margin	dimensionless
E_peak	Thermal energy of one peak event	kWh
E_peak_daily	Total daily peak thermal energy	kWh/day
3.3 Buffer Capacity Inputs
Symbol	Meaning	Unit
E_buffer_nominal	Nominal buffer energy capacity	kWh
E_buffer_usable	Usable buffer energy capacity	kWh
E_buffer_required	Required usable buffer capacity	kWh
SOC_buffer	Buffer state of charge	0–1
SOC_min	Minimum allowed state of charge	0–1
SOC_max	Maximum allowed state of charge	0–1
eta_buffer_charge	Buffer charge efficiency	0–1
eta_buffer_discharge	Buffer discharge efficiency	0–1
eta_buffer_roundtrip	Round-trip buffer efficiency	0–1
3.4 Water Buffer Inputs
Symbol	Meaning	Unit
m_buffer_water	Water mass in buffer	kg
V_buffer_water	Water volume in buffer	m³
Cp_water	Specific heat capacity of water	kJ/kg·K
rho_water	Water density	kg/m³
T_buffer_hot	Hot buffer temperature	°C
T_buffer_cold	Cold buffer temperature	°C
DeltaT_buffer	Usable buffer temperature swing	K
3.5 PCM Buffer Inputs
Symbol	Meaning	Unit
m_PCM	Phase-change material mass	kg
L_PCM	PCM latent heat	kJ/kg
T_PCM_melt	PCM phase-change temperature	°C
eta_PCM_use	Fraction of latent capacity actually usable	0–1
3.6 Solid Media Buffer Inputs
Symbol	Meaning	Unit
m_solid	Solid storage medium mass	kg
Cp_solid	Specific heat of solid medium	kJ/kg·K
DeltaT_solid	Usable temperature swing	K
rho_solid	Solid medium density	kg/m³
V_solid	Solid medium volume	m³
3.7 Loss Inputs
Symbol	Meaning	Unit
U_buffer	Overall heat transfer coefficient	W/m²·K
A_buffer	Buffer external surface area	m²
T_buffer_avg	Average buffer temperature	°C
T_ambient	Ambient temperature	°C
Q_loss_buffer	Heat loss from buffer	kW
t_storage	Storage duration	h
E_loss_buffer	Energy lost during storage	kWh
4. Default Constants
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³
seconds_per_hour = 3600 s/h
kWh_to_kJ = 3600 kJ/kWh
5. Heat Available for Buffer Charging

After higher-priority modules:

Q_buffer_in_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export

Constraint:

Q_buffer_in_available >= 0

If calculated value is negative:

Q_buffer_in_available = 0
6. Peak Energy Calculation

One peak event:

E_peak = Q_peak_excess × t_peak

Daily peak energy:

E_peak_daily = E_peak × N_peak_events

Required usable buffer capacity:

E_buffer_required =
E_peak × safety_factor_buffer

If the buffer must cover multiple peak events without full discharge between events:

E_buffer_required =
E_peak_daily × safety_factor_buffer

Constraints:

Q_peak_excess >= 0
t_peak >= 0
N_peak_events >= 0
safety_factor_buffer >= 1
7. Buffer Efficiency

Round-trip efficiency:

eta_buffer_roundtrip =
eta_buffer_charge × eta_buffer_discharge

Usable energy:

E_buffer_usable =
E_buffer_nominal × eta_buffer_discharge

Charged energy from incoming heat:

E_added_to_buffer =
E_heat_input × eta_buffer_charge

Delivered energy from stored heat:

E_delivered_from_buffer =
E_removed_from_buffer × eta_buffer_discharge

Constraints:

0 < eta_buffer_charge <= 1
0 < eta_buffer_discharge <= 1
0 < eta_buffer_roundtrip <= 1
8. Water Buffer Capacity

Thermal energy stored in water:

E_water_kJ =
m_buffer_water × Cp_water × DeltaT_buffer
E_water_kWh =
E_water_kJ / 3600

Water mass required:

m_buffer_water =
(E_buffer_required × 3600) / (Cp_water × DeltaT_buffer)

Water volume required:

V_buffer_water =
m_buffer_water / rho_water

Constraints:

DeltaT_buffer > 0
m_buffer_water > 0
V_buffer_water > 0
9. PCM Buffer Capacity

Latent energy stored in PCM:

E_PCM_kJ =
m_PCM × L_PCM × eta_PCM_use
E_PCM_kWh =
E_PCM_kJ / 3600

PCM mass required:

m_PCM =
(E_buffer_required × 3600) / (L_PCM × eta_PCM_use)

Constraints:

L_PCM > 0
0 < eta_PCM_use <= 1
m_PCM > 0

Temperature matching condition:

T_PCM_melt must be compatible with useful heat level.

Non-claim rule:

PCM storage capacity must not be claimed without verifying phase-change temperature, cycling stability, thermal conductivity, fire safety, and degradation.
10. Solid Media Buffer Capacity

Thermal energy stored in solid medium:

E_solid_kJ =
m_solid × Cp_solid × DeltaT_solid
E_solid_kWh =
E_solid_kJ / 3600

Solid mass required:

m_solid =
(E_buffer_required × 3600) / (Cp_solid × DeltaT_solid)

Solid volume required:

V_solid =
m_solid / rho_solid

Constraints:

Cp_solid > 0
DeltaT_solid > 0
rho_solid > 0
11. Charge Power Constraint

Maximum charge power:

Q_buffer_charge_max =
min(
    Q_buffer_in_available,
    Q_charge_equipment_max,
    Q_heat_exchanger_buffer_max,
    Q_flow_limit_buffer
)

Actual charge power:

Q_buffer_charge =
min(
    Q_buffer_in_available,
    Q_buffer_charge_max,
    Q_buffer_capacity_remaining / t_charge_available
)

Where:

Q_buffer_capacity_remaining =
E_buffer_nominal × (SOC_max - SOC_buffer)

Constraint:

Q_buffer_charge >= 0
SOC_buffer <= SOC_max
12. Discharge Power Constraint

Maximum discharge power:

Q_buffer_discharge_max =
min(
    Q_discharge_equipment_max,
    Q_heat_exchanger_discharge_max,
    Q_flow_limit_discharge,
    E_buffer_available_for_discharge / t_discharge_required
)

Available energy for discharge:

E_buffer_available_for_discharge =
E_buffer_nominal × (SOC_buffer - SOC_min)

Actual discharge power:

Q_buffer_discharge =
min(
    Q_buffer_discharge_max,
    Q_module_heat_demand_unmet
)

Constraint:

Q_buffer_discharge >= 0
SOC_buffer >= SOC_min
13. State of Charge Model

Buffer state of charge update over one time step:

SOC_buffer_next =
SOC_buffer
+ (Q_buffer_charge × eta_buffer_charge × Delta_t) / E_buffer_nominal
- (Q_buffer_discharge × Delta_t) / (E_buffer_nominal × eta_buffer_discharge)
- E_loss_buffer_step / E_buffer_nominal

Where:

Delta_t in h
Q in kW
E_buffer_nominal in kWh

Clamp condition:

SOC_buffer_next =
min(SOC_max, max(SOC_min, SOC_buffer_next))

Constraints:

0 <= SOC_min < SOC_max <= 1
SOC_min <= SOC_buffer <= SOC_max
14. Thermal Loss Model

First-order heat loss:

Q_loss_buffer =
(U_buffer × A_buffer × (T_buffer_avg - T_ambient)) / 1000

Where:

U_buffer in W/m²·K
A_buffer in m²
temperature difference in K
Q_loss_buffer in kW

Energy loss over storage time:

E_loss_buffer =
Q_loss_buffer × t_storage

Constraint:

Q_loss_buffer >= 0 when T_buffer_avg > T_ambient

If ambient is hotter than buffer:

Q_loss_buffer may become heat gain.
Thermal gain must be modeled separately if relevant.
15. Thermal Tail Interaction

Without buffer:

Q_tail_without_buffer =
Q_total - Q_WCT - Q_ADC - Q_WENV - Q_export

With buffer charging:

Q_tail_with_buffer =
Q_tail_without_buffer - Q_buffer_charge

Constraint:

Q_tail_with_buffer >= 0

Thermal tail reduction during charge:

thermal_tail_reduction_buffer =
Q_tail_without_buffer - Q_tail_with_buffer

Equivalent:

thermal_tail_reduction_buffer =
Q_buffer_charge

Non-claim rule:

Buffer charging reduces immediate thermal tail.
It does not eliminate heat over full cycle unless stored heat is later used by a useful process.
16. Useful Discharge Allocation

Buffer discharge priority:

1. W-CT unmet heat demand
2. A-DC unmet heat demand
3. W-env-DC unmet heat demand
4. heat export demand
5. auxiliary process demand
6. thermal tail

Allocation logic:

Q_buffer_remaining = Q_buffer_discharge

Q_buffer_to_WCT = min(Q_buffer_remaining, Q_WCT_unmet)
Q_buffer_remaining = Q_buffer_remaining - Q_buffer_to_WCT

Q_buffer_to_ADC = min(Q_buffer_remaining, Q_ADC_unmet)
Q_buffer_remaining = Q_buffer_remaining - Q_buffer_to_ADC

Q_buffer_to_WENV = min(Q_buffer_remaining, Q_WENV_unmet)
Q_buffer_remaining = Q_buffer_remaining - Q_buffer_to_WENV

Q_buffer_to_export = min(Q_buffer_remaining, Q_export_unmet)
Q_buffer_remaining = Q_buffer_remaining - Q_buffer_to_export

Q_buffer_to_aux = min(Q_buffer_remaining, Q_aux_unmet)
Q_buffer_remaining = Q_buffer_remaining - Q_buffer_to_aux

Q_buffer_to_tail = Q_buffer_remaining

Constraint:

Stored heat is useful only if discharged into a defined demand.
17. Reference Case A: 1 MW Data Center
17.1 Declared Inputs
P_IT = 1,000 kW

Q_peak_excess = 150 kW
t_peak = 2 h
N_peak_events = 1 per day
safety_factor_buffer = 1.20

eta_buffer_charge = 0.95
eta_buffer_discharge = 0.95

DeltaT_buffer = 20 K
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³

SOC_min = 0.10
SOC_max = 0.95
17.2 Peak Energy
E_peak = 150 × 2
E_peak = 300 kWh
E_buffer_required = 300 × 1.20
E_buffer_required = 360 kWh usable
17.3 Water Buffer Size
m_buffer_water =
(360 × 3600) / (4.186 × 20)

m_buffer_water =
1,296,000 / 83.72

m_buffer_water = 15,480 kg
V_buffer_water =
15,480 / 1000

V_buffer_water = 15.48 m³

Result:

A 360 kWh usable water buffer with 20 K temperature swing requires approximately 15.5 m³ of water before tank, insulation, and heat exchanger margins.
17.4 Charge Power Check

Assume:

Q_buffer_in_available = 200 kW
Q_charge_equipment_max = 180 kW
Q_heat_exchanger_buffer_max = 180 kW
Q_flow_limit_buffer = 250 kW
Q_buffer_charge_max =
min(200, 180, 180, 250)

Q_buffer_charge_max = 180 kW

Time to charge 360 kWh usable, ignoring losses:

t_charge =
E_buffer_required / (Q_buffer_charge_max × eta_buffer_charge)

t_charge =
360 / (180 × 0.95)

t_charge = 2.11 h
17.5 Discharge Check

Assume:

Q_module_heat_demand_unmet = 120 kW
Q_buffer_discharge_max = 150 kW
Q_buffer_discharge =
min(150, 120)

Q_buffer_discharge = 120 kW

Available delivery duration:

t_discharge =
E_buffer_required × eta_buffer_discharge / Q_buffer_discharge

t_discharge =
360 × 0.95 / 120

t_discharge = 2.85 h
18. Reference Case B: 10 MW Data Center
18.1 Declared Inputs

Scale from Case A:

P_IT = 10,000 kW

Q_peak_excess = 1,500 kW
t_peak = 2 h
N_peak_events = 1 per day
safety_factor_buffer = 1.20

eta_buffer_charge = 0.95
eta_buffer_discharge = 0.95

DeltaT_buffer = 20 K
Cp_water = 4.186 kJ/kg·K
rho_water = 1000 kg/m³

SOC_min = 0.10
SOC_max = 0.95
18.2 Peak Energy
E_peak = 1,500 × 2
E_peak = 3,000 kWh
E_buffer_required = 3,000 × 1.20
E_buffer_required = 3,600 kWh usable
18.3 Water Buffer Size
m_buffer_water =
(3,600 × 3600) / (4.186 × 20)

m_buffer_water =
12,960,000 / 83.72

m_buffer_water = 154,800 kg
V_buffer_water =
154,800 / 1000

V_buffer_water = 154.8 m³

Result:

A 3.6 MWh usable water buffer with 20 K temperature swing requires approximately 155 m³ of water before tank, insulation, and heat exchanger margins.
18.4 Charge Power Check

Assume:

Q_buffer_in_available = 2,000 kW
Q_charge_equipment_max = 1,800 kW
Q_heat_exchanger_buffer_max = 1,800 kW
Q_flow_limit_buffer = 2,500 kW
Q_buffer_charge_max =
min(2000, 1800, 1800, 2500)

Q_buffer_charge_max = 1800 kW

Time to charge 3.6 MWh usable, ignoring losses:

t_charge =
3600 / (1800 × 0.95)

t_charge = 2.11 h
18.5 Discharge Check

Assume:

Q_module_heat_demand_unmet = 1,200 kW
Q_buffer_discharge_max = 1,500 kW
Q_buffer_discharge =
min(1500, 1200)

Q_buffer_discharge = 1200 kW

Available delivery duration:

t_discharge =
3600 × 0.95 / 1200

t_discharge = 2.85 h
19. PCM Sensitivity Example

Assume:

E_buffer_required = 360 kWh
L_PCM = 200 kJ/kg
eta_PCM_use = 0.85

PCM mass:

m_PCM =
(360 × 3600) / (200 × 0.85)

m_PCM =
1,296,000 / 170

m_PCM = 7,624 kg

Result:

For 360 kWh usable storage, PCM mass is approximately 7.6 tonnes at L_PCM = 200 kJ/kg and eta_PCM_use = 0.85.

Scaling to 10 MW reference:

E_buffer_required = 3,600 kWh

m_PCM =
(3,600 × 3600) / (200 × 0.85)

m_PCM = 76,235 kg

Result:

For 3.6 MWh usable storage, PCM mass is approximately 76 tonnes under the same assumptions.

Interpretation rule:

PCM may reduce volume compared with water but may increase cost, thermal conductivity complexity, fire/safety requirements, and cycling validation requirements.
20. Buffer Loss Example

Assume 1 MW case:

U_buffer = 0.30 W/m²·K
A_buffer = 60 m²
T_buffer_avg = 50°C
T_ambient = 25°C
t_storage = 6 h

Heat loss:

Q_loss_buffer =
(0.30 × 60 × (50 - 25)) / 1000

Q_loss_buffer =
450 / 1000

Q_loss_buffer = 0.45 kW

Energy loss:

E_loss_buffer =
0.45 × 6

E_loss_buffer = 2.7 kWh

Loss fraction:

loss_fraction =
2.7 / 360

loss_fraction = 0.0075

Result:

Under declared assumptions, 6-hour storage loss is 0.75% of 360 kWh.
Actual loss depends on insulation, geometry, temperature, and ambient conditions.
21. Process Output Requirements

Thermal buffer calculation must output:

Q_buffer_in_available
Q_buffer_charge
Q_buffer_discharge
Q_buffer_charge_max
Q_buffer_discharge_max
Q_peak_excess
t_peak
N_peak_events
E_peak
E_peak_daily
E_buffer_required
E_buffer_nominal
E_buffer_usable
SOC_buffer
SOC_min
SOC_max
eta_buffer_charge
eta_buffer_discharge
eta_buffer_roundtrip
m_buffer_water
V_buffer_water
m_PCM
m_solid
Q_loss_buffer
E_loss_buffer
Q_tail_without_buffer
Q_tail_with_buffer
thermal_tail_reduction_buffer
Q_buffer_to_WCT
Q_buffer_to_ADC
Q_buffer_to_WENV
Q_buffer_to_export
Q_buffer_to_aux
Q_buffer_to_tail
22. Control Logic
INPUT:
    Q_captured
    Q_WCT
    Q_ADC
    Q_WENV
    Q_export
    SOC_buffer
    SOC_min
    SOC_max
    E_buffer_nominal
    module_unmet_heat_demands
    Q_tail_without_buffer

PROCESS:
    calculate Q_buffer_in_available

    IF Q_buffer_in_available > 0 AND SOC_buffer < SOC_max:
        calculate Q_buffer_charge_max
        charge buffer
        update SOC_buffer
        reduce immediate thermal tail

    ELSE IF Q_buffer_in_available > 0 AND SOC_buffer >= SOC_max:
        route excess heat to thermal tail

    IF module_unmet_heat_demand > 0 AND SOC_buffer > SOC_min:
        calculate Q_buffer_discharge_max
        discharge buffer by priority:
            W-CT
            A-DC
            W-env-DC
            heat export
            auxiliary process
        update SOC_buffer

    calculate buffer losses
    update SOC_buffer after losses

OUTPUT:
    updated SOC_buffer
    charged heat
    discharged heat
    thermal tail with buffer
    useful discharged heat allocation
    buffer full / empty status
23. Failure / Bypass Conditions
IF buffer_fault == true:
    isolate thermal buffer
    preserve primary data center cooling
    route Q_buffer allocation to thermal tail or other safe heat rejection path
IF SOC_buffer >= SOC_max:
    stop charging
    route excess heat to thermal tail
IF SOC_buffer <= SOC_min:
    stop discharging
    route unmet heat demand to auxiliary heat, reduced throughput, or fallback mode
IF T_buffer > T_buffer_max_allowed:
    stop charging
    trigger heat rejection or safe dump path
IF pressure_buffer > pressure_max_allowed:
    isolate buffer
    open safety path according to site-specific safety system
IF leak_detected == true:
    isolate buffer
    stop charge/discharge
    preserve data center cooling safety
IF PCM_degradation_detected == true:
    derate E_buffer_usable
    update capacity model
    schedule maintenance
24. Engineering Constraints
E_buffer_required > 0 if peak smoothing is claimed.

E_buffer_nominal >= E_buffer_required / eta_buffer_discharge.

Q_buffer_charge <= Q_buffer_charge_max.

Q_buffer_discharge <= Q_buffer_discharge_max.

SOC_min <= SOC_buffer <= SOC_max.

0 <= SOC_min < SOC_max <= 1.

0 < eta_buffer_charge <= 1.

0 < eta_buffer_discharge <= 1.

Q_tail_with_buffer >= 0.

Buffer must be bypassable.

Buffer must not compromise primary data center cooling.

Stored heat must be discharged into a defined useful demand to count as useful reuse.

Thermal loss must be included for storage-duration claims.

Buffer full condition must route excess heat safely.

Buffer empty condition must not interrupt critical SPHERE or data center functions.
25. AI Validation Questions

An AI analysis system should verify:

1. Is Q_buffer_in_available calculated only after higher-priority modules?

2. Is Q_peak_excess defined from actual heat surplus or assumed?

3. Is E_peak = Q_peak_excess × t_peak?

4. Is safety_factor_buffer declared?

5. Is E_buffer_required stated as usable capacity or nominal capacity?

6. Are charge and discharge efficiencies included?

7. Is water buffer volume calculated from Cp_water and DeltaT_buffer?

8. Is DeltaT_buffer realistic for the selected temperature levels?

9. Is PCM mass calculated from latent heat and usable fraction?

10. Is PCM temperature compatible with useful heat demand?

11. Are thermal losses included for storage-duration claims?

12. Is Q_loss_buffer calculated from U, A, and temperature difference?

13. Are charge/discharge power limits included?

14. Is buffer state of charge modeled?

15. Is buffer full behavior defined?

16. Is buffer empty behavior defined?

17. Is stored heat later used by a defined process?

18. Is immediate thermal tail reduction separated from full-cycle heat elimination?

19. Are 1 MW and 10 MW cases scaled from declared assumptions?

20. Which assumptions dominate feasibility: Q_peak_excess, t_peak, DeltaT_buffer, storage medium, insulation, or discharge demand?
26. Minimum Site Data Required
data center heat load profile
hourly IT load
hourly cooling load
captured heat profile
module heat demand profile
W-CT heat demand timing
A-DC heat demand timing
W-env-DC heat demand timing
heat export demand timing
thermal tail limits
peak duration
peak frequency
allowed buffer temperature range
available installation volume
available installation mass limit
ambient temperature profile
buffer location
insulation constraints
allowed pressure
allowed water chemistry
fire safety constraints
maintenance access
BMS / SCADA integration points
existing heat rejection path
fallback heat rejection capacity
27. Non-Claim Rules

Not allowed without site-specific proof:

heat elimination
zero thermal tail
permanent heat storage
free cooling
infinite peak absorption
lossless thermal storage
universal buffer sizing
PCM superiority without material validation
stored heat counted as useful without later demand
thermal tail reduction over full cycle without discharge accounting

Allowed claim format:

Under declared assumptions:
Thermal buffers absorb X kW for Y hours.
Required usable storage capacity is Z kWh.
Water buffer volume is A m³ at DeltaT = B K.
PCM mass is C kg at L_PCM = D kJ/kg.
Immediate thermal tail is reduced by E kW during charge.
Stored heat must later be discharged into defined useful demand or rejected as thermal tail.
28. Summary Formula Block
Q_buffer_in_available =
Q_captured
- Q_WCT
- Q_ADC
- Q_WENV
- Q_export

E_peak =
Q_peak_excess × t_peak

E_peak_daily =
E_peak × N_peak_events

E_buffer_required =
E_peak × safety_factor_buffer

eta_buffer_roundtrip =
eta_buffer_charge × eta_buffer_discharge

E_buffer_usable =
E_buffer_nominal × eta_buffer_discharge

E_water_kJ =
m_buffer_water × Cp_water × DeltaT_buffer

E_water_kWh =
E_water_kJ / 3600

m_buffer_water =
(E_buffer_required × 3600) / (Cp_water × DeltaT_buffer)

V_buffer_water =
m_buffer_water / rho_water

E_PCM_kJ =
m_PCM × L_PCM × eta_PCM_use

E_PCM_kWh =
E_PCM_kJ / 3600

m_PCM =
(E_buffer_required × 3600) / (L_PCM × eta_PCM_use)

E_solid_kJ =
m_solid × Cp_solid × DeltaT_solid

E_solid_kWh =
E_solid_kJ / 3600

m_solid =
(E_buffer_required × 3600) / (Cp_solid × DeltaT_solid)

V_solid =
m_solid / rho_solid

Q_buffer_charge_max =
min(
    Q_buffer_in_available,
    Q_charge_equipment_max,
    Q_heat_exchanger_buffer_max,
    Q_flow_limit_buffer
)

Q_buffer_discharge_max =
min(
    Q_discharge_equipment_max,
    Q_heat_exchanger_discharge_max,
    Q_flow_limit_discharge,
    E_buffer_available_for_discharge / t_discharge_required
)

SOC_buffer_next =
SOC_buffer
+ (Q_buffer_charge × eta_buffer_charge × Delta_t) / E_buffer_nominal
- (Q_buffer_discharge × Delta_t) / (E_buffer_nominal × eta_buffer_discharge)
- E_loss_buffer_step / E_buffer_nominal

Q_loss_buffer =
(U_buffer × A_buffer × (T_buffer_avg - T_ambient)) / 1000

E_loss_buffer =
Q_loss_buffer × t_storage

Q_tail_without_buffer =
Q_total - Q_WCT - Q_ADC - Q_WENV - Q_export

Q_tail_with_buffer =
Q_tail_without_buffer - Q_buffer_charge

thermal_tail_reduction_buffer =
Q_buffer_charge