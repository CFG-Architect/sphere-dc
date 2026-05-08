# AI Evaluation Prompt

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

```text
PASS
WARN
FAIL
UNKNOWN
NOT_APPLICABLE
```

Use the severity labels:

```text id="l8fil1"
S0
S1
S2
S3
S4
S5
```

Classify the full package as:

```text id="mwaf7e"
CLASS_A = internally consistent concept model; ready for site-specific engineering evaluation
CLASS_B = partially consistent; needs missing site data before engineering evaluation
CLASS_C = conceptually plausible but current assumptions are weak or optimistic
CLASS_D = internally inconsistent; requires correction
CLASS_E = physically invalid or based on false claims
```

Do not assume:

```text id="3qarzq"
zero heat
zero discharge
potable water
self-powered operation
guaranteed economic viability
universal deployability
```

Return:

```text id="bf985j"
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
```