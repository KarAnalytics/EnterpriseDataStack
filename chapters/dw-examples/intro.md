# DW Examples

Concepts stick when you see them applied. This chapter walks through concrete star-schema designs for three familiar business processes — retail sales, airline operations, and healthcare visits — so you can pattern-match the next time you face a blank page.

## Example 1 — Retail sales

A retailer wants to analyze sales across products, stores, customers, and time.

**Grain:** one row per product line on a sales transaction at a store on a date.

**Dimensions:**
- `dim_date` — calendar attributes, fiscal periods, holidays
- `dim_product` — product, brand, sub-category, category, department
- `dim_store` — store, city, state, region, format (mall / standalone / online)
- `dim_customer` — loyalty tier, demographics, acquisition channel (SCD Type 2)
- `dim_promotion` — promotion code, discount %, start/end date

**Fact table:** `fact_sales`
- FKs: `date_sk`, `product_sk`, `store_sk`, `customer_sk`, `promotion_sk`
- Degenerate: `pos_transaction_id`
- Measures: `units_sold`, `gross_amount`, `discount_amount`, `net_amount`, `cost_amount`, `margin_amount`

**Example question answered:** *"Net revenue by department, region, and month for the last 24 months, with a promotion flag comparison."* — a `SUM` over `net_amount`, grouping by three dimension attributes, filtered by a date range. No joins beyond the star.

## Example 2 — Airline operations

An airline wants a unified view of flights, bookings, and loyalty activity.

**Multiple fact tables, conformed dimensions:**
- `fact_bookings` — grain: one booking
- `fact_flight_segments` — grain: one passenger × one flight leg
- `fact_loyalty_activity` — grain: one loyalty event (earn, redeem)

**Conformed dimensions** across all three:
- `dim_date` (role-playing: `booking_date`, `flight_date`)
- `dim_passenger` (SCD Type 2 — tier changes over time)
- `dim_airport` (role-playing: `origin`, `destination`)
- `dim_aircraft`
- `dim_flight_number`

**Key design choices:**
- `dim_airport` plays multiple roles (origin, destination) via views
- A **bridge table** handles multi-leg itineraries linking a single booking to many flight segments
- A **junk dimension** bundles low-cardinality flags (`is_refundable`, `fare_class_group`, `booking_channel`)

**Example question:** *"Average on-time percentage by route, aircraft type, and month-of-year."* — joins `fact_flight_segments` with `dim_airport` (origin + destination roles), `dim_aircraft`, and `dim_date`.

## Example 3 — Healthcare visits

A hospital system wants to analyze patient visits and outcomes.

**Grain:** one row per patient visit.

**Dimensions:**
- `dim_date` (role-playing: `admit_date`, `discharge_date`)
- `dim_patient` (SCD Type 2, with sensitive attributes flagged for governance)
- `dim_provider`
- `dim_facility`
- `dim_insurance`

**Bridge table:**
- `bridge_diagnosis_group` — each visit links to many diagnoses via a diagnosis group key; weights allocate charges across diagnoses where the business requires it

**Fact table:** `fact_visit`
- FKs: `admit_date_sk`, `discharge_date_sk`, `patient_sk`, `provider_sk`, `facility_sk`, `insurance_sk`, `diagnosis_group_sk`
- Measures: `length_of_stay_days`, `total_charges`, `insurance_paid`, `patient_paid`, `readmitted_30d_flag`

**Governance note:** PHI fields in `dim_patient` are access-controlled; analytics users get a view with masked or tokenized values unless they have explicit clearance. This is Chapter 9 (data governance) showing up in the physical design.

## Patterns you'll keep seeing

- **Date dimension is always first** — build it once, pre-populate 20+ years of rows, share across every fact
- **Role-playing dimensions show up whenever a fact has multiple dates**
- **Conformed dimensions are what lets you compare across business processes**
- **Junk dimensions tame flag explosions**
- **SCD Type 2 is the default for customer / patient / employee dimensions**
- **Governance constraints shape the physical model, not just access policies**

## Learning outcomes

- Walk through Kimball's four-step process on a new business scenario
- Recognize when to introduce bridge, junk, and mini-dimensions
- Identify conformed dimensions across multiple fact tables
- Spot governance touchpoints in a warehouse design
- Translate a business question into a star-schema query

