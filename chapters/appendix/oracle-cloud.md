# Setting up Oracle Cloud

All SQL in this book runs on an **Oracle Autonomous Database** in Oracle Cloud, through a browser-based SQL Worksheet. There is nothing to install, and the **Always Free** tier is enough for every example and exercise.

## 1. Create an Oracle Cloud account

**If you are taking the course**, you will receive an invitation email from Oracle Cloud at your university address. Follow the link to cloud.oracle.com and:

1. Create a **cloud account name** (a short name for your tenancy; you will type it every time you sign in, so write it down).
2. Use the exact university email address the invitation was sent to, not an alias.
3. **Do not enter any payment information.** The class account does not need it. If you are asked for a credit card, stop and contact your instructor: your email is probably not on the class list yet.

**If you are reading on your own**, sign up for an [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/) account. Oracle may ask for a card to verify your identity, but resources marked **Always Free** are not charged.

Then:

4. Set up **secure verification** (multi-factor authentication). The Oracle Mobile Authenticator app is the easiest option.
5. Sign in. New accounts may come with trial credits for paid services. When they expire, the account continues as a free account, and Always Free resources keep working.

## 2. Create an Autonomous Database

1. From the Oracle Cloud home page, choose **Create an Autonomous Data Warehouse** (under Quickstarts), or open the menu and go to **Oracle Database > Autonomous Database > Create Autonomous Database**.
2. Fill in the form:
   - **Display name**: something you will recognize, such as `ADW` + your initials + `01`. Do not include confidential information.
   - **Database name**: letters and numbers only, starting with a letter, at most 14 characters. The display name without spaces works.
   - **Workload type**: Data Warehouse.
   - **Always Free**: switch this **on**. The CPU and storage fields then lock to the free configuration (at the time of writing, 1 OCPU with 20 GB of storage). If you leave it off, the database uses paid resources and consumes credits.
   - **Administrator credentials**: the username is always **ADMIN**. Choose a password of 12 to 30 characters with at least one lowercase letter, one uppercase letter and one number; it cannot contain a double quote (`"`) or the word "admin". **Write down the username and password**: you will use them every time.
   - Leave the advanced options at their defaults.
3. Click **Create**. Provisioning takes a few minutes. Check that the instance is marked **Always Free**.
4. When the status turns green (**Available**), your database server is running.

## 3. Open the SQL Worksheet

1. On the database's page, click **Database actions** and choose **SQL**.
2. Sign in as **ADMIN** with your password.
3. Take the short tour of the interface the first time.
4. **Bookmark the page**, so you can go straight to the SQL Worksheet next time.

The SQL Worksheet is part of **Database Actions** (SQL Developer Web). Other tools there, such as the **Data Modeler** (see [Advanced SQL Queries](../sql-advanced/intro.md)), are in the menu at the top left.

:::{warning} Keep your database alive
An Always Free database that is not used for about a week is stopped automatically, and one that stays unused for a long time (around 90 days) can be deleted by Oracle. Sign in every few days while you are working through the book. If your database is stopped, start it again from its page in the cloud console.
:::

## 4. Sample data

Every Autonomous Database includes Oracle's read-only sample schemas. This book uses the **SH (Sales History)** schema for many query examples; see [The SH Sample Schema](sh-schema.md). Tables you create yourself (for example, in the SQL hands-on exercises) live in your ADMIN schema.

## Troubleshooting

| Problem | What to do |
|---|---|
| Asked for a credit card (course account) | Stop and contact your instructor; do not enter card details. |
| Cannot find your database | Check the **compartment** selector on the Autonomous Database page; the database is usually in the root compartment. Also check that the **region** at the top right is the one where you created it. |
| Database Actions will not open | The database may be stopped. Start it from its page and wait for the status to turn green. |
| Forgot the ADMIN password | On the database's page, choose **More actions > Administrator password** to reset it. |
