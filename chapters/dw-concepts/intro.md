# Data Warehousing Concepts

The databases of [Part III](../er-model/intro.md) are built to run the business: record each sale, update each account, ship each order. They are not built to *understand* the business: to compare this year with last year across every store, product and channel. That is the job of a **data warehouse**. This chapter explains what a data warehouse is, why organizations build one, the two classic philosophies for building it, and the related ideas of data marts, data lakes, operational data stores and virtual warehouses.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Define a data warehouse and explain why OLTP databases are not enough for analytics.
2. Compare OLTP and OLAP systems.
3. Contrast Inmon's (top-down) and Kimball's (bottom-up) approaches.
4. Distinguish data warehouses, data marts (dependent and independent), data lakes, operational data stores and virtual warehouses.
:::

## What a data warehouse is

A **data warehouse (DW)** is an integrated, enterprise-wide collection of high-quality information from different databases, used to support management decision making and performance management.

Each part of the definition matters:

- **Integrated**: data from many source systems, reconciled to common definitions (one meaning of "customer", one product code).
- **Enterprise-wide**: it crosses departmental boundaries.
- **High-quality**: it is cleaned and profiled on the way in (see [Part II](../data-processing/intro.md)).
- **For decision making**: it is designed for questions and reports, not for recording transactions.

## Why we need data warehouses

Operational databases, or **online transaction processing (OLTP)** systems, are concerned with the CRUD operations on data: create, read, update and delete. Each OLTP system handles the everyday running of one part of the enterprise, such as point of sale, payroll or inventory. They are usually designed independently of each other, and it is hard for them to share information.

A data warehouse instead supports **online analytical processing (OLAP)**: preparing and querying data for analysis. It is a critical component of **business intelligence (BI)**. Organizations build data warehouses to:

- **consolidate** information from many sources in one place;
- **improve query performance** for analytical questions;
- **separate** research and decision support from the operational systems, so a heavy report does not slow down the checkout;
- provide a **foundation** for data mining, visualization, advanced reporting and OLAP tools.

Typical applications: banks assessing credit risk, insurers searching for fraud, medical researchers analyzing outcomes, and every retailer that asks questions like these:

- Who are my customers, and what products are they buying?
- Which customers are most likely to go to a competitor?
- Which customers have the highest and lowest margins?
- Which promotions have the biggest impact on revenue?
- What is the most effective distribution channel?
- What will new products or services do to revenue and margins?

### OLTP versus OLAP

| | OLTP | OLAP |
|---|---|---|
| Purpose | run the business: process transactions | understand the business: analyze |
| Data | current | current and historical |
| Modeling | entities and relationships (ER model, normalized) | facts and dimensions (star schema, denormalized) |
| Operations | read, write and update, many small transactions | read only, with periodic bulk loads |
| Queries | record-oriented ("this customer's balance") | aggregate ("sales by region by quarter") |
| Storage | relational tables | relational tables or data cubes |
| Users | operational staff | managers and analysts |

The design difference is the most visible one. An operational database is designed with the ER model and normalized (Part III); a data warehouse is designed as a **star schema** of facts and dimensions, deliberately denormalized for fast, simple queries ([Data Warehouse Design](../dw-design/intro.md)).

### An example BI application

A retail company collects purchase data in the databases of all its stores and loads it into a central data warehouse every night. Analysts can then:

- see a daily summary of all sales;
- analyze all stores in a region together;
- drill down from a region to a store, from a category to a specific product, or to the sales of a specific supplier's products;
- analyze trends over months and years;

and managers use the results when negotiating with suppliers and advertisers, and when deciding on discounts and product bundles.

## Two philosophies: Inmon and Kimball

Two pioneers defined the two classic ways to build a data warehouse.

**Bill Inmon's approach (top-down).** The data warehouse is one part of the overall business intelligence system. An enterprise has **one** central data warehouse, stored in **third normal form (3NF)**, integrated from all the source systems. **Data marts** for individual departments are then built *from* the warehouse, usually in dimensional form. The design starts with the enterprise data model and works down to the departments.

**Ralph Kimball's approach (bottom-up).** The data warehouse is the collection of all the **data marts** in the enterprise. Information is always stored in the **dimensional model** (star schemas). Data marts are built one business process at a time, and they are tied together by **conformed dimensions**: shared dimension tables (such as one Date dimension and one Product dimension) used by every mart. The warehouse grows from the departments up.

| | Inmon | Kimball |
|---|---|---|
| Direction | top-down | bottom-up |
| Central store | one enterprise warehouse in 3NF | the set of dimensional data marts |
| Data marts | dependent, fed from the warehouse | the building blocks of the warehouse |
| First results | slower: the enterprise model comes first | faster: one business process at a time |
| Integration | in the central normalized warehouse | through conformed dimensions |


:::{figure} images/inmon_kimball.png
:label: fig-inmon-kimball
:alt: Two architectures side by side. Inmon: source systems feed an enterprise data warehouse in 3NF, which feeds dependent data marts. Kimball: source systems feed dimensional stars for sales, inventory and finance, tied together by conformed dimensions.
Inmon's top-down and Kimball's bottom-up architectures. Source: {download}`inmon_kimball.drawio <images/inmon_kimball.drawio>`.
:::

In practice many organizations combine them: a normalized integration layer feeding dimensional marts that users query.

## Data marts and data lakes

A **data mart** is a data warehouse with a limited scope, designed for one function or department (sales, finance, marketing). There are two kinds:

- **Dependent data marts** draw their data from a central data warehouse (Inmon's approach).
- **Independent data marts** are standalone systems that draw data directly from operational systems, external sources, or both. They are quick to build, but several independent marts can each define "revenue" differently.

A **data lake** is a central repository of data stored in its raw, unprocessed format: files, logs, documents, images and database extracts. Data is stored first and given structure when it is read ("schema on read", as in [Spark](../spark/intro.md)), whereas a warehouse structures data before loading it.

A typical data flow combines them:

:::{figure} images/typical_dataflow.png
:label: fig-typical-dataflow
:alt: A row of five colored boxes connected by arrows: external data sources, data lake, operational databases, data marts and data warehouses, with a note that Inmon builds the warehouse first and then the marts from it.
A typical data flow, built bottom-up as in Kimball's approach. In Inmon's alternative, the data warehouse is built first and the data marts are created from it.
:::

## Operational data stores

An **operational data store (ODS)** is an integrated, frequently refreshed copy of operational data, often used as a staging area, with query facilities. It suits analysis that needs data close to real time, such as today's orders across all channels. More complex, historical analysis is still done in the data warehouse.

## Virtual data warehouses and data marts

**Virtualization** uses middleware to create a logical (virtual) data warehouse or data mart that holds no physical data of its own. It provides a single point of access to a set of underlying data stores, and the data is pulled from them only at query time. The simplest virtual data mart is a set of **SQL views** over the operational tables:

```sql
CREATE VIEW sales_by_month AS
SELECT t.calendar_month_desc AS month,
       p.prod_category       AS category,
       SUM(s.amount_sold)    AS sales
FROM   sh.sales s
JOIN   sh.times t    ON s.time_id = t.time_id
JOIN   sh.products p ON s.prod_id = p.prod_id
GROUP  BY t.calendar_month_desc, p.prod_category;
```

Users query `sales_by_month` like a table, but every query recomputes it from the sources. Virtual warehouses are quick to set up and always current; physical warehouses are faster for heavy queries and keep history even when sources change.

:::{note} Think about it
A regional hospital network runs separate systems for admissions, the pharmacy, the laboratory and billing. Its leaders want to know the average length of stay, cost and readmission rate by diagnosis, physician and month, across all its hospitals. Why can't they simply query the four operational systems? List three problems a data warehouse would solve for them.
:::

## Common mistakes

- **Querying the OLTP system for analytics.** Long aggregate queries compete with transactions and can slow down the business.
- **Treating the warehouse as a copy of the database.** Without integration, cleaning and a dimensional design, it is just a second copy of the problems.
- **Building independent marts that disagree.** Two marts with different definitions of "customer" produce two answers to the same question. Conform the dimensions.
- **Calling a data lake a warehouse.** Raw data in a lake still needs processing before it is fit for decisions.

:::{warning} Check the AI
Ask a GenAI assistant "Inmon or Kimball?" and you will usually get a balanced table, sometimes with errors such as "Kimball does not use a central warehouse" (Kimball's warehouse is the integrated set of conformed data marts) or "Inmon's warehouse is a star schema" (it is normalized; the marts built from it are dimensional). More importantly, the right choice depends on your organization's sources, skills and urgency, which the assistant does not know. Use its comparison as a checklist of questions to ask, not as the decision.
:::

## Practice

**1. OLTP or OLAP?** (a) Recording a hotel reservation. (b) Average occupancy by hotel, month and booking channel over five years. (c) Checking whether room 412 is free tonight. (d) Which loyalty tier generates the most revenue per guest.

:::{note} Answer
:class: dropdown
(a) OLTP. (b) OLAP. (c) OLTP. (d) OLAP.
:::

**2. Which structure?** (a) Marketing wants its own small warehouse, fed nightly from the corporate warehouse. (b) A data science team stores years of raw website click logs, sensor files and scanned documents for future use. (c) Customer service needs a near-real-time, integrated view of today's orders from all channels. (d) Finance builds a standalone reporting database directly from the general ledger system.

:::{note} Answer
:class: dropdown
(a) A dependent data mart. (b) A data lake. (c) An operational data store. (d) An independent data mart.
:::

**3. Inmon or Kimball?** A mid-sized company wants a sales dashboard within three months, then plans to add inventory and finance analysis over the next two years. Which approach fits better, and what must it do to avoid inconsistent marts later?

:::{note} One possible answer
:class: dropdown
Kimball's bottom-up approach delivers the sales mart first. To keep later marts consistent, it must design conformed dimensions (Date, Product, Store, Customer) from the start and reuse them in the inventory and finance marts.
:::

**4. A virtual mart.** In your Oracle Cloud account, create the `sales_by_month` view above (in your own schema) and query it for one category. Then explain one advantage and one disadvantage of answering this question with a view instead of a physical summary table.

## Key terms

data warehouse, OLTP, OLAP, business intelligence, Inmon (top-down), Kimball (bottom-up), conformed dimension, data mart, dependent and independent data marts, data lake, schema on read, operational data store, virtual data warehouse, SQL view, drill down

## Further reading

- Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit* (3rd ed.), chapter 1.
- Inmon, W. H. (2005). *Building the Data Warehouse* (4th ed.). Wiley.
