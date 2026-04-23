# Data Governance

Data governance is the set of policies, standards, roles, and controls that make an organization's data **trustworthy, lawful, and fit for purpose**. It is the least glamorous topic in this book and arguably the most important — every scandal you've read about in the last decade (data breaches, discriminatory algorithms, opaque AI decisions) is, at its root, a governance failure.

## Why governance matters more now

Modern regulations put real teeth behind what used to be best practices:

- **GDPR** (EU) — right to access, rectify, erase; data minimization; explicit consent
- **CCPA / CPRA** (California) — consumer rights similar to GDPR
- **HIPAA** (US healthcare) — PHI handling and breach notification
- **SOX** (financial reporting) — auditability of data that feeds financial statements
- **FERPA** (US education), **GLBA** (US financial), **PCI DSS** (payment cards), sector-specific rules in every industry

Fines for non-compliance are measured in percentages of global revenue. Governance is no longer a nice-to-have.

## Pillars of data governance

A mature governance program covers six pillars, and they touch every stage of the EDM pipeline:

| Pillar | Concern | Example control |
|---|---|---|
| **Ethics** | Is using this data the right thing to do? | Review board for new AI models |
| **Security** | Can the wrong person read this data? | Role-based access, encryption at rest and in transit |
| **Privacy** | Are individual rights respected? | Consent tracking, PII tagging, data minimization |
| **Fairness** | Are outputs free from unjust bias? | Disparate-impact audits on ML models |
| **Interpretability** | Can we explain decisions made from this data? | Model cards, lineage tracking |
| **Assurance & regulation** | Can we prove we're compliant? | Audit logs, retention policies, DPIA documentation |

## Roles and responsibilities

Governance is a team sport with defined roles:

- **Data owner** — accountable for a domain of data (e.g., the VP of Sales owns customer data)
- **Data steward** — day-to-day responsibility for quality, definitions, and issue resolution
- **Data custodian** — IT role that operates the storage / pipelines
- **Data protection officer (DPO)** — required by GDPR for large-scale processing of sensitive data
- **Data governance committee** — cross-functional forum that sets policy

## Key artifacts

- **Data catalog** — a searchable inventory of datasets with owners, definitions, and sensitivity tags
- **Business glossary** — shared definitions of terms (what exactly is a "customer"?)
- **Data lineage** — where a field came from and what transformations it went through
- **Data classification scheme** — typically something like public / internal / confidential / restricted
- **Data retention policy** — how long data is kept and how it is disposed of
- **Access control matrix** — who can do what, mapped to roles

## Governance in the EDM pipeline

Governance is the horizontal slice across Chapter 2's pipeline:

```
Planning → Ingestion → Integration → Wrangling → DBMS → Warehousing
     └──────────────── GOVERNANCE ────────────────┘
```

Every stage generates governance obligations. Ingestion decides what gets captured; integration decides how identities are resolved; warehousing decides who gets to query what; ML decides whose outcomes get predicted.

## Governance and AI

Generative AI and ML systems multiply governance obligations:

- **Training-data provenance** — were you licensed to train on this data?
- **Inference-time privacy** — do your prompts leak PII to a third-party API?
- **Output auditability** — can you explain why the model said what it said?
- **Drift and retraining** — is yesterday's fair model still fair today?

Expect your career to include increasing amounts of AI governance work.

## Learning outcomes

- Name the major regulations relevant to a given industry
- Describe the six governance pillars and give a concrete control for each
- Identify the owner, steward, and custodian roles and what each is responsible for
- Point to the governance obligations at each stage of the EDM pipeline
- Articulate why governance is especially acute for AI / ML systems

