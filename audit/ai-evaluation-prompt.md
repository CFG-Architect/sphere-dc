AI Evaluation Prompt — SPHERE-DC
Evaluation Scope
Analyze the attached SPHERE-DC technical package as a concept-level engineering model.
Do not evaluate it as a marketing text.
Do not require site-specific data to classify the package — missing site data is an expected and declared condition of this package, not a defect.
Evaluate internal consistency, physical validity, and quality of the declared assumptions — not deployment readiness.

Documents to Evaluate Together
01-thermal-core.md
02-assumptions-and-variables.md
03-w-ct.md
04-a-dc.md
05-w-env-dc.md
06-thermal-buffers.md
07-heat-export.md
08-auxiliary-power.md
09-integrated-balance-1mw-10mw.md
10-ai-validation-checklist.md

Check

internal consistency
heat balance
water balance
electric balance
thermodynamic limits
mass conservation
double counting
non-claim compliance
missing site data
dominant weak points


Validation Labels
PASS          = internally consistent under declared assumptions
WARN          = plausible but depends on missing site-specific data
FAIL          = internally inconsistent, physically invalid, or overclaimed
UNKNOWN       = cannot evaluate because required data is missing
NOT_APPLICABLE = item does not apply to the declared case

Severity Labels
S0 = informational
S1 = minor issue
S2 = important assumption gap
S3 = major feasibility risk
S4 = critical invalidation risk
S5 = impossible / physically invalid / false claim

Feasibility Classification
Classify the full package using exactly one of the following:
CLASS_A = internally consistent concept model; ready for site-specific engineering evaluation
          Requires: all balances internally consistent, no S4/S5 failures,
          non-claim rules preserved, dominant risks quantified

CLASS_B = partially consistent; needs missing site data before engineering evaluation
          Requires: internal logic consistent, no physically impossible claims,
          some P0/P1 site data missing, reference cases plausible but not site-validated

CLASS_C = conceptually plausible but current assumptions are weak or optimistic
          Requires: physically plausible concept, but key assumptions unvalidated,
          dominant modules depend on site conditions, feasibility uncertain

CLASS_D = internally inconsistent; requires correction before further evaluation
          Requires: internal contradictions, double counting, major missing calculations,
          or S4 failures present

CLASS_E = physically invalid or based on false claims
          Requires: violation of physical constraints, impossible efficiency claims,
          self-powered data center claim, zero discharge without reject accounting,
          or S5 failures present

Do Not Assume
zero heat
zero discharge
potable water
self-powered operation
guaranteed economic viability
universal deployability

Return
1.  Executive technical verdict
2.  System-level feasibility status
3.  Module-level validation table (status + severity per module)
4.  Heat balance validation
5.  Water balance validation
6.  Electric balance validation
7.  Site-data gaps (P0 = required before any feasibility claim)
8.  Overclaim / non-claim audit
9.  Dominant sensitivity variables
10. Critical failure modes
11. Recommended next calculations
12. Final AI feasibility classification (CLASS_A / B / C / D / E)
    + primary reason
    + top 5 technical risks
    + top 5 missing site data items
    + top 5 most sensitive variables

Notes on Reference Cases
The 1 MW and 10 MW reference cases in the technical documents use illustrative module demand values.
These values are set to demonstrate allocation arithmetic, not derived from process calculations.
Do not treat the specific kW figures as validated engineering demands.
Evaluate whether the allocation logic and formulas are internally consistent — not whether the example numbers reflect a specific real site.

Notes on Auxiliary Power
Auxiliary power output from low-grade DC waste heat is expected to be small relative to total heat input.
This is a declared structural limitation of the system, not an overclaim.
The relevant check is: does the auxiliary power calculation respect the Carnot limit and declare net output honestly?
Do not flag small auxiliary output as a defect — flag it only if it is misrepresented as significant or self-powering.
