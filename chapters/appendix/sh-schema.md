# The SH Sample Schema

**SH (Sales History)** is one of the sample schemas that come with every Oracle Autonomous Database (the other common one is SSB, the Star Schema Benchmark). Many SQL examples in this book query it, because it is real-sized: almost a million sales rows.

## The business behind the data

SH belongs to a fictional retailer that does a high volume of business and runs statistical reports to support decisions. Many of the reports are time-based and look at past trends: annual, quarterly, monthly and weekly sales by product. The company also analyzes:

- sales by **distribution channel** (direct sales, internet, partners, ...);
- the impact of **promotions** on sales;
- sales by **geography** (country, subregion and region).

The company loads data into its data warehouse regularly, so SH is organized as a **star schema**: one large fact table of sales surrounded by dimension tables. See [Data Warehouse Design](../dw-design/intro.md).

## Tables

In your SQL Worksheet the tables are in the `SH` schema, so you write `SH.SALES`, `SH.CUSTOMERS`, and so on. The main columns are listed below; primary keys are underlined.

| Table | Main columns |
|---|---|
| SALES (fact) | PROD_ID, CUST_ID, TIME_ID, CHANNEL_ID, PROMO_ID, QUANTITY_SOLD, AMOUNT_SOLD |
| CUSTOMERS | {u}`CUST_ID`, CUST_FIRST_NAME, CUST_LAST_NAME, CUST_GENDER, CUST_YEAR_OF_BIRTH, CUST_CITY, CUST_STATE_PROVINCE, COUNTRY_ID, CUST_INCOME_LEVEL, CUST_CREDIT_LIMIT, CUST_EMAIL |
| COUNTRIES | {u}`COUNTRY_ID`, COUNTRY_NAME, COUNTRY_SUBREGION, COUNTRY_REGION |
| PRODUCTS | {u}`PROD_ID`, PROD_NAME, PROD_DESC, PROD_SUBCATEGORY, PROD_CATEGORY, PROD_LIST_PRICE, PROD_MIN_PRICE |
| CHANNELS | {u}`CHANNEL_ID`, CHANNEL_DESC, CHANNEL_CLASS |
| PROMOTIONS | {u}`PROMO_ID`, PROMO_NAME, PROMO_SUBCATEGORY, PROMO_CATEGORY, PROMO_COST, PROMO_BEGIN_DATE, PROMO_END_DATE |
| TIMES | {u}`TIME_ID`, DAY_NAME, CALENDAR_MONTH_DESC, CALENDAR_YEAR, FISCAL_YEAR, and many more calendar and fiscal columns |

SALES has no primary key of its own. Its CUST_ID, PROD_ID, TIME_ID, CHANNEL_ID and PROMO_ID columns are foreign keys to the tables of the same name, and CUSTOMERS.COUNTRY_ID is a foreign key to COUNTRIES. The sales cover the calendar years 1998 to 2001.

## Getting to know the data

A few queries worth running before you use SH (try them in your SQL Worksheet):

```sql
-- how big is each table?
SELECT COUNT(*) FROM SH.SALES;
SELECT COUNT(*) FROM SH.CUSTOMERS;

-- what does a sale look like?
SELECT * FROM SH.SALES FETCH FIRST 10 ROWS ONLY;

-- which channels and product categories exist?
SELECT * FROM SH.CHANNELS;
SELECT DISTINCT PROD_CATEGORY FROM SH.PRODUCTS;
```

Real sample data has quirks, and finding them is good practice for [data profiling](../data-profiling/intro.md). In SH, for example, QUANTITY_SOLD is 1 on every row, and only a handful of the hundreds of promotions are ever used in a sale. Always look at the data before trusting a total.

## Reference

- Oracle, [Sample Schemas: Sales History (SH)](https://docs.oracle.com/en/database/oracle/oracle-database/19/comsc/introduction-to-sample-schemas.html), with full table descriptions.
