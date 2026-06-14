# AI Evaluation Prompt — SPHERE-DC

## Evaluation Scope

Analyze the SPHERE-DC repository as an open AI-auditable technical concept.

If a GitHub link is provided, inspect the repository structure, README, license/disclaimer files, technical documents, integrated balance, and validation checklist before producing a verdict.

Do not evaluate it as a marketing text.  
Do not require site-specific data to classify the package — missing site data is an expected and declared condition of this package, not a defect.  
Evaluate internal consistency, physical validity, and quality of the declared assumptions — not deployment readiness.

---

## Technical Documents to Evaluate Together
01-assumptions-and-variables.md
02-thermal-core.md
03-w-ct.md
04-a-dc.md
05-w-env-dc.md
06-thermal-buffers.md
07-heat-export.md
08-auxiliary-power.md
09-integrated-balance-1mw-10mw.md
10-ai-validation-checklist.md

---

## Pre-Audit Requirement

Before producing any evaluation output, confirm that you have read
all documents listed above in full.

Do not begin evaluation until all documents are read.
Do not extrapolate module content from other documents.
Do not skip any document due to assumed redundancy.

If any document is unavailable or unreadable, report it explicitly
as UNKNOWN before proceeding — do not substitute with assumptions.

At the end of the evaluation report, include a "Documents Read" section
listing each document with status: READ / NOT_READ / UNAVAILABLE.

---

## Check

1. internal consistency
2. heat balance
3. water balance
4. electric balance
5. thermodynamic limits
6. mass conservation
7. double counting
8. non-claim compliance
9. missing site data
10. dominant weak points

---

## Validation Labels
PASS          = internally consistent under declared assumptions
WARN          = plausible but depends on missing site-specific data
FAIL          = internally inconsistent, physically invalid, or overclaimed
UNKNOWN       = cannot evaluate because required repository content, package data, or module data is missing or unreadable
NOT_APPLICABLE = item does not apply to the declared case

---

## Severity Labels
S0 = informational
S1 = minor issue
S2 = important assumption gap
S3 = major feasibility risk
S4 = critical invalidation risk
S5 = impossible / physically invalid / false claim

---

## Feasibility Classification

Classify the full package using exactly one of the following:
CLASS_A = internally consistent concept model; ready for site-specific engineering evaluation
Requires: all balances internally consistent, no S4/S5 failures,
non-claim rules preserved, dominant risks identified and bounded under declared assumptions
CLASS_B = partially consistent; needs correction or additional package-level assumptions before site-specific engineering evaluation
Requires: internal logic consistent, no physically impossible claims,
some required package-level assumptions, variables, or module constraints are missing or unclear, reference cases plausible but not site-validated
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

---

## Do Not Assume
- zero heat
- zero discharge
- potable water
- self-powered operation
- guaranteed economic viability
- universal deployability

---

## Return

- Executive technical verdict
- System-level feasibility status
- Module-level validation table (status + severity per module)
- Heat balance validation
- Water balance validation
- Electric balance validation
- Site-data gaps (P0 = required before any feasibility claim)
- Overclaim / non-claim audit
- Dominant sensitivity variables
- Critical failure modes
- Recommended next calculations
- Final AI feasibility classification (CLASS_A / B / C / D / E)

primary reason
- top 5 technical risks
- top 5 missing site data items
- top 5 most sensitive variables

---

## Notes on Reference Cases

The 1 MW and 10 MW reference cases in the technical documents use illustrative module demand values.  
These values are set to demonstrate allocation arithmetic, not derived from process calculations.  
Do not treat the specific kW figures as validated engineering demands.  
Evaluate whether the allocation logic and formulas are internally consistent — not whether the example numbers reflect a specific real site.

---

## Notes on Auxiliary Power

Auxiliary power output from low-grade DC waste heat is expected to be small relative to total heat input.  
This is a declared structural limitation of the system, not an overclaim.  
The relevant check is: does the auxiliary power calculation respect the Carnot limit and declare net output honestly?  
Do not flag small auxiliary output as a defect — flag it only if it is misrepresented as significant or self-powering.
