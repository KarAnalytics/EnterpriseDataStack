# Data Governance

Clean, well-designed data can still do harm: when it is collected without consent, shared with the wrong people, used for a purpose nobody agreed to, or left unprotected. **Data ethics**, **data privacy** and **data governance** are about handling data rightly as well as well. This chapter introduces the ethical concerns, the main US privacy laws, and how organizations assign responsibility for their data.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain what data ethics is and why organizations need it.
2. Identify ethical concerns at each stage of data's life, and recognize common violations.
3. Name the major US data privacy laws, what each protects, and how GDPR differs.
4. Describe data governance, its roles (owners, stewards, steering committees) and its practices.
:::

## Data ethics

**Data ethics** is the work of systematizing, defending and recommending concepts of right and wrong conduct in relation to data, in particular personal data. An organization that takes data ethics seriously:

- reduces internal conflicts and strengthens a sense of common purpose among its people;
- protects vulnerable populations and individuals who could be harmed by intentional or unintentional misuse or breaches;
- protects its reputation and the trust it has earned from customers and the public.

### Where ethical concerns arise

Ethical questions follow data through its life:

| Stage | Example questions |
|---|---|
| **Collection** | Did people agree to this? Are we collecting more than we need? |
| **Storage** | Who can access it? Is it secure? How long will we keep it? |
| **Sharing, reuse and replicability** | May we share it with partners, or use it for a new purpose? Can others verify our results? |
| **Re-identification and consent** | Could "anonymous" data be linked back to individuals? Did consent cover this use? |

Re-identification deserves special attention. Removing names is rarely enough: a combination of zip code, birth date and gender identifies a large share of the US population, so a dataset can be "anonymous" and still expose people when joined with another dataset.

### Common data ethics violations

- Collecting personal data without consent.
- Using personal data for unauthorized purposes.
- Failing to protect personal data from unauthorized access or disclosure.
- Discriminating against individuals based on their personal data.
- Failing to keep personal data accurate and up to date.
- Not allowing individuals to access and control their personal data.

## Data privacy

**Data privacy** concerns the relationship between the collection and dissemination of data, technology, the public's expectation of privacy, and the legal and political issues around them. The United States has no single comprehensive federal privacy law. Instead, privacy is protected by many laws, each covering a sector or a type of data, plus a growing number of state laws.

The laws you are most likely to meet:

| Law | Protects | Key idea |
|---|---|---|
| **HIPAA**, Health Insurance Portability and Accountability Act (1996) | patients' health information held by health care providers, insurers and their business associates | sensitive health information may not be disclosed without the patient's consent or knowledge, except as the law allows |
| **FERPA**, Family Educational Rights and Privacy Act (1974) | student education records at schools that receive federal funding | students (and parents of minors) control access to their records and can review and correct them |
| **COPPA**, Children's Online Privacy Protection Act (1998) | children under 13 online | websites and apps need verifiable parental consent before collecting children's personal information |
| **GLBA**, Gramm-Leach-Bliley Act (1999) | consumers' financial information | financial institutions must explain their information-sharing and safeguard customer data |
| **FCRA**, Fair Credit Reporting Act (1970) | consumer credit reports | accuracy, fairness and limits on who may see a credit report |

Many other federal laws protect particular kinds of data, among them the Privacy Act of 1974 (records held by federal agencies), the Electronic Communications Privacy Act, the Video Privacy Protection Act, the Telephone Consumer Protection Act, the Genetic Information Nondiscrimination Act and the HITECH Act (which strengthened HIPAA). The **Federal Trade Commission (FTC)** enforces against unfair and deceptive data practices across industries. States add their own laws, led by California's Consumer Privacy Act (CCPA, amended by the CPRA).

The European Union's **General Data Protection Regulation (GDPR)**, in force since 2018, takes a different approach: one comprehensive law that protects the personal data of people in the EU, wherever the company processing it is located. It requires a lawful basis for processing, gives individuals rights to access, correct and erase their data, and requires data minimization and breach notification. Many US companies follow it because they have European customers. In the US, the combination of FTC enforcement, sector laws and state laws adds up to some, but not all, of GDPR's protections.

:::{note} For BSAN 726 and 727 students
FERPA applies to you directly: your grades, submissions and other education records are protected. The same law is why course materials never show real students' work without permission, and why group-project data from a client must stay within the project.
:::

## Data governance

In many organizations, nobody is formally responsible for data. That does not work: data that nobody owns cannot be monitored or managed responsibly. **Data governance** is a business-led, continuous process of improving data for the benefit of all its stakeholders. It establishes who is formally accountable for the data in each enterprise system, and the policies and controls they apply.

Governance is relevant to every stage of the EDM pipeline (see [EDM Foundations](../edm-foundations/intro.md)) and to every decision-making level.

### Roles

| Role | Responsibility |
|---|---|
| **Data owner** | a business leader accountable for a data domain (for example, customer data): decides who may use it and for what |
| **Data steward** | the day-to-day caretaker of a domain: maintains definitions, monitors quality, resolves issues |
| **Data governance steering committee** | senior representatives who set policy, prioritize, and settle disputes across domains |
| **Data custodian** | usually IT: operates the systems that store the data, implementing security, backup and access controls |


:::{figure} images/governance_roles.png
:label: fig-governance-roles
:alt: A diagram: the steering committee sets policies for the data owner, who delegates care to the data steward and decides access implemented by the data custodian; both serve the data users.
Data governance roles and how they relate. Source: {download}`governance_roles.drawio <images/governance_roles.drawio>`.
:::

These are often additional roles taken on by programmers, analysts and managers rather than full-time jobs.

### Good governance in practice

- **Establish ownership** and responsibilities for each data domain.
- **Develop policies and procedures**: classification (public, internal, confidential, restricted), retention, acceptable use, sharing.
- **Implement security measures**: role-based access, encryption, audit logs.
- **Ensure data quality**: shared definitions, profiling, and quality rules (see [Data Profiling](../data-profiling/intro.md)).
- **Monitor data usage**: who accessed what, and was the use authorized?
- **Report** on governance activities to leadership.

## Worked example: governing a student success project

A university analytics team wants to predict which first-year students may need academic support, using enrollment records, grades, financial aid status and learning management system activity.

| Question | Governance answer |
|---|---|
| Which laws apply? | FERPA covers the education records. Financial aid data has additional federal restrictions. If any data came from the health center, it would be governed separately and should not be used. |
| Who owns the data? | The registrar (enrollment, grades), the financial aid office and the IT learning-systems team each own their domain; each must approve the use. |
| What is the minimum data needed? | Course grades, credits attempted and LMS logins may be enough; home address, ethnicity and detailed aid amounts may not be needed. Collect only what is justified. |
| How is it protected? | Analysts work with a student ID replaced by a project key; the key file stays with the registrar's data steward. Access is limited to the named team. |
| What are the ethical risks? | A model could label students unfairly, for example by learning patterns tied to income or background. Predictions should trigger an offer of support, never a penalty, and should be checked for unequal error rates across groups. |
| When does it end? | The project dataset is deleted after the analysis period, as the retention policy specifies. |

## Common mistakes

- **Assuming "anonymous" means safe.** Combinations of ordinary fields can re-identify people.
- **Collecting everything "just in case".** Data you do not need is risk without benefit.
- **Treating governance as IT's job.** Owners are business leaders; IT implements their decisions.
- **Reusing data for a new purpose without checking consent.**
- **Thinking compliance equals ethics.** A use can be legal and still harmful or unfair.

:::{warning} Check the AI
Two cautions. First, what you paste into a GenAI assistant may leave your control: never paste customer records, student data, health information or a client's confidential data into a tool that your organization has not approved for it. Use synthetic or sample data to ask your question instead. Second, assistants summarize laws confidently and sometimes wrongly, for example stating that GDPR applies only to EU citizens (it protects people in the EU) or attributing a rule to the wrong law. Use their answers as a starting point, and confirm legal questions with the official text, your organization's privacy office or legal counsel.
:::

## Practice

**1. Which law?** Name the US law most directly involved: (a) a fitness app for children asks for an 11-year-old's email address; (b) a hospital's billing vendor emails a spreadsheet of patient diagnoses to the wrong client; (c) a parent asks to see their 16-year-old's high school records; (d) a bank shares customer account data with a marketing affiliate.

:::{note} Answer
:class: dropdown
(a) COPPA. (b) HIPAA (the vendor is a business associate of the hospital). (c) FERPA. (d) GLBA.
:::

**2. Ethics violations.** Which common violation does each describe? (a) A store uses loyalty-card purchases to infer pregnancies and sends baby-product ads before customers have told anyone. (b) An insurer keeps charging a higher rate because an old, corrected traffic violation is still in its records. (c) A website will not let users download or delete the data it holds about them.

:::{note} Answer
:class: dropdown
(a) Using personal data for purposes the customer did not authorize. (b) Failing to keep personal data accurate and up to date. (c) Not allowing individuals to access and control their personal data.
:::

**3. Roles.** For the Kansas City 311 call data, who might be the data owner, the data steward and the data custodian?

:::{note} One possible answer
:class: dropdown
Owner: the director of the department that runs the 311 call center. Steward: a call-center data analyst who maintains the call categories and monitors quality. Custodian: the city's IT department, which runs the call-tracking system and the open data portal.
:::

**4. A governance checklist.** Pick a dataset from your work or a course project. Write a one-page governance note: the data owner, the laws or policies that apply, the minimum data needed, how it will be protected, the main ethical risk, and when it will be deleted.

## Key terms

data ethics, re-identification, consent, data privacy, HIPAA, FERPA, COPPA, GLBA, FCRA, FTC, CCPA, GDPR, data minimization, data governance, data owner, data steward, data custodian, governance steering committee, data classification, retention policy

## Further reading

- US Department of Health and Human Services, [Summary of the HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html).
- US Department of Education, [FERPA](https://studentprivacy.ed.gov/ferpa).
- Federal Trade Commission, [Children's Online Privacy Protection Rule (COPPA)](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa).
- European Commission, [Data protection in the EU](https://commission.europa.eu/law/law-topic/data-protection_en).
- DAMA International, *DAMA-DMBOK* (2nd ed.), chapter on data governance.
