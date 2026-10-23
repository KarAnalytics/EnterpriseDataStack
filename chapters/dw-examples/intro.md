# Data Warehouse Examples

Dimensional modeling works in almost every industry, because almost every organization has business processes that produce measurable events. This chapter surveys typical star schema designs across fourteen business processes, from retail to insurance to web analytics, and closes with the process teams follow to build such designs. Treat the designs as templates: a real organization adapts them to its own processes and data sources.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Recognize the business process, grain, dimensions and facts in a range of industry examples.
2. Distinguish transaction, periodic snapshot and accumulating snapshot fact tables.
3. Adapt a template design to a new organization.
4. Describe the steps of a dimensional modeling project.
:::

## Three kinds of fact table

The examples use three kinds of fact table, which differ in what one row represents:

| Kind | One row per | Example | Typical facts |
|---|---|---|---|
| **Transaction** | event, at the moment it happens | one item scanned at checkout | quantity, amount (additive) |
| **Periodic snapshot** | entity per regular period | each product in each warehouse at the end of each day | quantity on hand, balance (often semi-additive) |
| **Accumulating snapshot** | process instance, updated as it moves through milestones | one order, from order to shipment to delivery to payment | dates of each milestone, durations between them |

## Industry templates

Each table row is a starting point, not a finished design: the grain, the dimension attributes and the facts all depend on what the organization actually records.

### Retail and supply chain

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **Retail sales** | one product on one point-of-sale transaction | Date, Product, Store, Promotion, Cashier, Payment method; transaction number (degenerate) | quantity, sales amount, discount, cost, gross profit |
| **Inventory** | each product in each store or warehouse each day (periodic snapshot) | Date, Product, Store/Warehouse | quantity on hand (semi-additive), quantity received and shipped, inventory value |
| **Procurement** | one line on a purchase order, or a snapshot per order line through its milestones | Date (several roles: ordered, received, invoiced), Product, Vendor, Contract terms, Buyer | quantity and amount ordered, received and invoiced; lead time |
| **Order management** | one line on a customer order | Date (order, requested ship), Customer, Product, Sales rep, Deal/discount, Channel; order number (degenerate) | quantity, gross and net amount, discount |

### Finance and accounting

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **General ledger (accounting)** | one journal entry line, or each account each month (snapshot) | Date/Fiscal period, Account (chart of accounts hierarchy), Organization, Cost center | debit and credit amounts; period-end balance (semi-additive) |
| **Account management (banking)** | each account each month (periodic snapshot) | Month, Account, Customer/Household, Product (checking, savings, loan), Branch, Status | balance (semi-additive), interest paid or earned, fees, transaction counts |

### Services and transportation

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **Telecommunications** | one call or data session (call detail record), or each customer each billing month | Date, Time of day, Customer, Rate plan, Originating and terminating location, Service type | duration, data volume, charge |
| **Airline flight activity** | one passenger on one flight segment | Date (flight and booking dates), Passenger/Frequent flyer, Flight, Origin and destination airport (role-playing), Fare class, Booking channel | segment miles, fare revenue, taxes, upgrade flag counts |
| **Rental car transactions** | one rental agreement (accumulating snapshot from reservation to return) | Date (reservation, pickup, return), Customer, Vehicle class, Pickup and return location, Rate plan | rental days, miles driven, base and extra charges |

### Education and health care

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **Student applications** | one application (accumulating snapshot through submitted, complete, admitted, enrolled) | Date of each milestone, Applicant, Program, Term, Recruiting source | days between milestones; counts admitted and enrolled (0/1 flags) |
| **Medical claim billing** | one line item on a claim | Date (service, claim, payment), Patient, Provider, Payer, Diagnosis, Procedure, Facility | billed, allowed and paid amounts, patient responsibility |

Health care designs must also follow privacy law ([Data Governance](../data-governance/intro.md)): patient dimensions in analytical warehouses are often de-identified.

### Insurance

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **Policy transactions** | one transaction on a policy (create, change coverage, renew, cancel) | Date, Policyholder, Policy, Coverage, Covered item, Agent, Transaction type | written premium change, coverage limit change |
| **Claims processing** | one claim (accumulating snapshot through reported, assigned, settled, closed), or each claim transaction | Date of each milestone, Claimant, Policy, Coverage, Adjuster, Cause of loss | reserve amount, paid amount, days open |

### Digital

| Process | Grain | Dimensions | Facts |
|---|---|---|---|
| **Clickstream** | one page event (click), or one visitor session | Date, Time of day, Visitor, Page, Session (degenerate or its own dimension), Referring source, Device | seconds on page, page views, items added to cart, conversion flags |

Clickstream is the extreme case of a fine grain: a busy website produces millions of rows per day, which is one reason clickstream data is often stored in a [data lake](../dw-concepts/intro.md) or on a [big data platform](../hadoop/intro.md) before it is summarized into the warehouse.

## Patterns across the examples

- **The date dimension is everywhere**, and often plays several roles in one fact table (order, ship, payment dates).
- **Customer, product and location dimensions recur**, and should be conformed: the same Customer dimension in sales, service and billing marts lets analysts combine them.
- **Processes with milestones** (orders, claims, applications, rentals) suit accumulating snapshots, which make "how long does each step take?" an easy query.
- **Balances** (inventory, accounts, ledgers) suit periodic snapshots, and are semi-additive.
- **Document numbers** (transaction, order, claim, ticket numbers) become degenerate dimensions.

## The dimensional modeling process

A dimensional design is built with the business, not just for it. A typical project:

1. **Preparation.** Identify the business sponsor, the participants (business experts, source system experts, modelers), and the business requirements: the questions people need to answer.
2. **High-level model.** In workshops, choose the business process, declare the grain, and sketch the dimensions and facts on a single page (a "bubble chart" of the star).
3. **Detailed design.** Define every dimension attribute and fact, its source, its transformation rules, and how each dimension handles change (slowly changing dimension types).
4. **Review and validation.** Check the design against real source data (profiling) and walk through it with business users, using their actual questions.
5. **Finalize and document.** Record the design, the source-to-target mapping, and open issues; hand over to the ETL team ([Part VI](../etl-procedure/intro.md)).

Throughout, the four core decisions of [Data Warehouse Design](../dw-design/intro.md) recur: **business process, grain, dimensions, facts**.

## Common mistakes

- **Copying a template unchanged.** A template's dimensions may not exist in your sources, and your business may have dimensions the template lacks.
- **Modeling a department instead of a process.** "Marketing's warehouse" mixes processes; model "campaign responses" and "web sessions" as processes.
- **Mixing snapshot and transaction grains** in one fact table.
- **Ignoring privacy** in customer, patient and student dimensions.

:::{warning} Check the AI
GenAI assistants know these templates well, because they are widely published, and will produce a star schema for "an airline" or "a hospital" instantly. That is useful as a first draft and dangerous as a final design. The assistant cannot know your sources, your business rules or your privacy constraints, and it tends to add every dimension that might conceivably apply. For each table in its answer, ask: which source system provides this, at this grain? If nothing does, it belongs in a requirements discussion, not in the schema.
:::

## Practice

**1. Which kind of fact table?** (a) Each bank account's balance at the end of every day. (b) Each item scanned at a self-checkout. (c) Each mortgage application, with the dates it was received, appraised, approved and closed.

:::{note} Answer
:class: dropdown
(a) Periodic snapshot. (b) Transaction. (c) Accumulating snapshot.
:::

**2. Adapt a template.** A city library system wants to analyze e-book loans: which titles are borrowed, by which types of patrons, through which app, and how long each loan lasts. Starting from the retail sales template, propose the grain, dimensions and facts.

:::{note} One possible answer
:class: dropdown
Grain: one e-book loan. Dimensions: Date (loan date, and return date in a second role), Title (author, subject, publisher, format), Patron (patron type, age band, home branch; no names), App/Platform. Facts: loan count (1) and loan days (filled at return; an accumulating snapshot if the row is updated at return).
:::

**3. Conformed dimensions.** A hospital network has three marts: admissions, pharmacy and billing. Which dimensions should be conformed across all three, and why?

:::{note} One possible answer
:class: dropdown
Date, Patient (de-identified), Facility and Provider at least, and Diagnosis where it applies. Conforming them lets analysts follow one patient's stay across admission, medication and billing, and compare facilities consistently.
:::

**4. Design your own.** Choose a business process you know from work (or from a course project). Write the four core decisions (process, grain, dimensions, facts), choose the fact table kind, and draw the star schema in draw.io.

## Key terms

transaction fact table, periodic snapshot, accumulating snapshot, industry template, conformed dimension, milestone, de-identification, dimensional modeling process, business requirements, bubble chart, source-to-target mapping

## Further reading

- Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit* (3rd ed.). The case-study chapters cover each of the industries above in depth, with complete diagrams.
