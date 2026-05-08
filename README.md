# SPHERE-DC

**SPHERE-DC** is an AI-evaluable technical concept for integrating data center heat, water, air, and auxiliary energy flows into a structured engineering system.

This repository does not present a finished commercial product.  
It presents a technical concept, system logic, engineering assumptions, validation constraints, and an AI-auditable documentation package.

## Core Idea

A data center should not be treated only as a source of waste heat, dirty water, air load, and residual technical outputs.

SPHERE-DC treats these outputs as interconnected flows:

```text
heat → capture → allocation → useful process / export / buffer / tail
water → recovery → treatment → clean technical water / reject
air → filtration → humidity control → condensate / load
energy → limited auxiliary support power
```

## Start Here

1. Read the concept article:  
   [SPHERE-DC Article](article/sphere-dc-article.md)

2. Review the technical documentation package:  
   [Technical Documentation Index](#technical-documentation)

3. Run an AI validation using:  
   [AI Evaluation Prompt](audit/ai-evaluation-prompt.md)

## Technical Documentation

1. [Thermal Core](docs/01-thermal-core.md)
2. [Assumptions and Variables](docs/02-assumptions-and-variables.md)
3. [W-CT Module](docs/03-w-ct.md)
4. [A-DC Module](docs/04-a-dc.md)
5. [W-env-DC Module](docs/05-w-env-dc.md)
6. [Thermal Buffers](docs/06-thermal-buffers.md)
7. [Heat Export](docs/07-heat-export.md)
8. [Auxiliary Power](docs/08-auxiliary-power.md)
9. [Integrated Balance: 1 MW and 10 MW](docs/09-integrated-balance-1mw-10mw.md)
10. [AI Validation Checklist](docs/10-ai-validation-checklist.md)

## What This Is

```text id="7vhnbu"
engineering concept skeleton
system balance framework
AI-verifiable feasibility scaffold
anti-overclaim protocol
```

## What This Is Not

```text id="l27co0"
not a final engineering design
not a vendor-ready implementation package
not a commercial roadmap
not a construction-ready specification
not a claim of zero heat
not a claim of zero discharge
not a claim of self-powered data centers
not a claim of potable water production
```

## Recommended AI Audit Prompt

Use this prompt with the article and all technical documents:

```text id="g6ds84"
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
```

## Expected Status

The current package should be treated as:

```text id="mecdak"
concept-level engineering model
ready for independent AI audit
ready for site-specific professional evaluation
not ready for direct deployment
```

## Rights Notice

See [NOTICE.md](NOTICE.md).