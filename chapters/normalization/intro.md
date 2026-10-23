# Normalization

A good ER model, carefully mapped to tables, usually produces a sound database. But designs are not always that careful, and analysts constantly inherit spreadsheets and tables that store the same fact over and over. **Normalization** is the tool for validating and improving a logical design: it finds the redundancy and removes it, one well-defined step at a time.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain why redundancy causes insertion, deletion and modification anomalies.
2. Identify functional dependencies, and classify them as full, partial or transitive.
3. Check whether a relation is valid (1NF), and normalize it to 2NF and 3NF.
4. Keep the links between split relations with foreign keys (or mapping relations).
:::

## Why normalize

Normalization validates and improves a logical design so that it avoids unnecessary duplication, or **redundancy**, of data, and lets users insert, update and delete rows without creating problems. Redundancy is not just wasteful. If every employee row repeats the company's phone number, then changing that number means updating every employee's row.

The problems that redundancy causes are called **anomalies**:

- An **insertion anomaly**: you cannot record one fact until you have an unrelated fact to record with it.
- A **deletion anomaly**: deleting one fact accidentally deletes another.
- A **modification (update) anomaly**: changing one fact requires changing many rows, and missing one leaves the data contradicting itself.

A good conceptual design and a proper ER-to-relational conversion will automatically produce well-normalized tables. Normalization catches what the design missed, and it is the tool you need for data that never had a design.

## Functional dependencies

Normalization is driven by **functional dependencies (FDs)**. Attribute B is functionally dependent on attribute A, written **A → B**, if each value of A determines exactly one value of B. In an employee table, `EmployeeID → EmployeeName`: once you know the ID, the name is fixed. The attribute on the left is the **determinant**.

Three kinds of dependency matter:

- A **full dependency**: a non-key attribute depends on the *whole* primary key. In an order-line table keyed by (OrderID, ProductID), the quantity ordered depends on both.
- A **partial dependency**: a non-key attribute depends on only *part* of a composite primary key. The order date depends on OrderID alone.
- A **transitive dependency**: a non-key attribute depends on another non-key attribute, and so only indirectly on the key. Write it as a chain: OrderID → CustomerID → CustomerName.

In real life you may or may not be given the functional dependencies. Usually you must infer them from business knowledge: ask what the rules are, rather than guessing from a few rows of sample data. Two employees in a sample might share a department, but that does not mean `EmployeeName → Department`.

## The normalization steps

| Step | Result | What you do |
|---|---|---|
| 1 | **First normal form (1NF)** | Make the relation valid: one value per cell (remove multivalued attributes) and a valid primary key. |
| 2 | **Second normal form (2NF)** | Remove partial dependencies: every non-key attribute depends on the *entire* primary key. |
| 3 | **Third normal form (3NF)** | Remove transitive dependencies: every non-key attribute depends *only* on the primary key. |

Each step splits relations and adds foreign keys where necessary, so that the split tables can still be joined back together. 
:::{figure} images/nf_steps.png
:label: fig-nf-steps
:alt: Four boxes in a row: unnormalized, 1NF, 2NF and 3NF, with the step that leads to each.
The normalization steps. Source: {download}`nf_steps.drawio <images/nf_steps.drawio>`.
:::

A relation whose primary key is a single column is automatically in 2NF, because there is no "part" of the key for anything to depend on.

There are higher normal forms (Boyce-Codd, fourth and fifth), but most corporate databases are normalized to 3NF. Data warehouses deliberately go the other way and add planned redundancy, called **denormalization**, to make analytical queries simpler and faster; see [Data Warehouse Design](../dw-design/intro.md).

## Worked example: a farmers' market

A farmers' market manager tracks stall bookings in a spreadsheet, one row per vendor:

| VendorID | VendorName | VendorPhone | Bookings |
|---|---|---|---|
| V01 | Prairie Honey Co. | 785-555-0141 | May 2: S04 (zone A, $45); May 9: S04 (zone A, $45); May 16: S11 (zone B, $30) |
| V02 | Wakarusa Greens | 785-555-0178 | May 2: S11 (zone B, $30); May 9: S07 (zone B, $30) |
| V03 | Flint Hills Bakery | 785-555-0102 | May 9: S11 (zone B, $30) |

The business rules: each vendor books at most one stall per market day; each stall belongs to one zone; the daily rate is set per zone.

### Step 1: is the relation valid? (1NF)

Check two things: does each row have **one value per cell**, and does the relation have a **valid primary key**? Here the Bookings cell holds several bookings, a multivalued attribute, so the relation is not valid.

Resolve the multivalued attribute by giving each booking its own row, with one value per cell. One vendor now appears on several rows, so VendorID alone no longer identifies a row; the primary key becomes (VendorID, MarketDate).

> **BOOKING** ({u}`VendorID`, {u}`MarketDate`, VendorName, VendorPhone, StallNo, StallZone, ZoneRate)

| VendorID | MarketDate | VendorName | VendorPhone | StallNo | StallZone | ZoneRate |
|---|---|---|---|---|---|---|
| V01 | 2026-05-02 | Prairie Honey Co. | 785-555-0141 | S04 | A | 45 |
| V01 | 2026-05-09 | Prairie Honey Co. | 785-555-0141 | S04 | A | 45 |
| V01 | 2026-05-16 | Prairie Honey Co. | 785-555-0141 | S11 | B | 30 |
| V02 | 2026-05-02 | Wakarusa Greens | 785-555-0178 | S11 | B | 30 |
| V02 | 2026-05-09 | Wakarusa Greens | 785-555-0178 | S07 | B | 30 |
| V03 | 2026-05-09 | Flint Hills Bakery | 785-555-0102 | S11 | B | 30 |

No multivalued attributes and a valid primary key: the relation is now in 1NF. It still shows all three anomalies:

- **Modification:** if the zone B rate rises to $35, four rows must change. Miss one and the market has two zone B rates.
- **Insertion:** a new stall, S15 in zone C, cannot be recorded until some vendor books it, and a vendor who has registered but not yet booked cannot be recorded at all (MarketDate is part of the key and cannot be null).
- **Deletion:** if Flint Hills Bakery cancels its only booking, deleting that row also deletes the bakery's name and phone number.

### Functional dependencies

From the business rules:

- VendorID → VendorName, VendorPhone
- VendorID, MarketDate → StallNo
- StallNo → StallZone
- StallZone → ZoneRate


:::{figure} images/booking_dependencies.png
:label: fig-booking-dependencies
:alt: The seven BOOKING columns in a row with arrows: VendorID to VendorName and VendorPhone (partial), the whole key to StallNo (full), and StallNo to StallZone to ZoneRate (transitive).
The functional dependencies of the BOOKING relation in 1NF.
:::

### Step 2: remove partial dependencies (2NF)

Which attributes depend on only part of the key (VendorID, MarketDate)?

- VendorID → VendorName, VendorPhone is a **partial** dependency.
- StallNo depends on both VendorID and MarketDate (which stall depends on who booked *and* on which day), so it is a **full** dependency and stays.

Split the partially dependent attributes into a relation keyed by the part of the key they depend on, and keep that part behind as a foreign key:

> 1. **VENDOR** ({u}`VendorID`, VendorName, VendorPhone)
> 2. **BOOKING** ({u}`VendorID`, {u}`MarketDate`, StallNo, StallZone, ZoneRate)
>    FK (VendorID) ref VENDOR

### Step 3: remove transitive dependencies (3NF)

Which non-key attributes depend on other non-key attributes? In BOOKING there is a chain:

- VendorID, MarketDate → StallNo → StallZone → ZoneRate

Split each transitive dependency into its own relation, keyed by its determinant. Don't forget to keep the determinant behind as a **foreign key to the new parent table**, or you can no longer join the data back together:

1. **VENDOR** ({u}`VendorID`, VendorName, VendorPhone)
2. **ZONE** ({u}`StallZone`, ZoneRate)
3. **STALL** ({u}`StallNo`, StallZone)
   FK (StallZone) ref ZONE
4. **BOOKING** ({u}`VendorID`, {u}`MarketDate`, StallNo)
   FK (VendorID) ref VENDOR, FK (StallNo) ref STALL

Each fact now lives in one place. A rate change touches one row of ZONE; a new stall or a new vendor can be added before any booking; cancelling a booking deletes only the booking.

### Foreign key or mapping relation?

Splitting out a transitive dependency creates a one-to-many relationship (one stall, many bookings), and, as in [The Relational Model](../relational-model/intro.md), there are two ways to keep the link. The solution above adds the foreign key StallNo to BOOKING (option 1). Option 2 removes StallNo from BOOKING and adds a mapping relation instead:

> **BOOKING** ({u}`VendorID`, {u}`MarketDate`)
> **BOOKING_STALL** ({u}`VendorID`, {u}`MarketDate`, StallNo)
> FK (VendorID, MarketDate) ref BOOKING, FK (StallNo) ref STALL

Both are in 3NF. Option 1 needs fewer tables and joins; option 2 mirrors the "one table per relationship" style. Either way, the important thing is that the link survives the split.

### Beyond the mechanics

- The rule that a stall is booked by at most one vendor per day makes (StallNo, MarketDate) unique as well. In SQL you would enforce that with a `UNIQUE` constraint on BOOKING.
- If the market needs to know what each vendor actually *paid*, even after rates change, add an `AmountCharged` column to BOOKING. That is not redundancy: the amount charged on May 2 is a fact about that booking, while ZoneRate is the *current* price.

## Common mistakes

- **Inferring dependencies from sample data.** Ten rows can suggest a pattern that the business does not guarantee. Ask what the rules are.
- **Moving a full dependency.** An attribute that depends on the whole composite key (like StallNo above) stays where it is in step 2.
- **Splitting without keeping the link.** Every split must leave a foreign key (or a mapping relation) behind.
- **Stopping at "it has a primary key".** A relation can have a perfectly good key and still be only in 1NF.
- **Normalizing away history.** A price, address or rate *at the time of a transaction* belongs with the transaction.

:::{warning} Check the AI
GenAI assistants are good at the mechanics of normalization and weak at the business rules behind them. Asked to normalize the market spreadsheet, an assistant might return

> BOOKING (VendorID, MarketDate, StallNo, ZoneRate), "in 3NF because every column depends on the booking"

and sound completely confident. Test the claim with a question about the data: *can two bookings in the same zone have different rates?* Under this market's rules they cannot, so ZoneRate depends on the stall's zone, not on the booking, and the table repeats the rate on every booking in the zone. Ask the same question of every non-key attribute, and check that each split keeps a foreign key behind. If the assistant's answer depends on a rule you never gave it, it guessed.
:::

## Practice

**1. Single-column key.** A tutoring center logs sessions in one table: **SESSION** ({u}`SessionID`, SessionDate, TutorID, TutorName, TutorDept, StudentID, StudentName). Each session has one tutor and one student. What normal form is the table in? Normalize it to 3NF.

:::{note} Answer
:class: dropdown
The primary key is a single column, so there are no partial dependencies and the table is in 2NF. It is not in 3NF because of two transitive dependencies: SessionID → TutorID → TutorName, TutorDept and SessionID → StudentID → StudentName. In 3NF:

1. **TUTOR** ({u}`TutorID`, TutorName, TutorDept)
2. **STUDENT** ({u}`StudentID`, StudentName)
3. **SESSION** ({u}`SessionID`, SessionDate, TutorID, StudentID)
   FK (TutorID) ref TUTOR, FK (StudentID) ref STUDENT
:::

**2. Composite key.** A car-sharing club records rentals as **RENTAL** ({u}`CarID`, {u}`RentalDate`, CarModel, ModelSeats, MemberID, MemberName, MilesDriven), with these dependencies: CarID → CarModel; CarModel → ModelSeats; CarID, RentalDate → MemberID, MilesDriven; MemberID → MemberName. Give one anomaly, then normalize to 3NF.

:::{note} Answer
:class: dropdown
Anomaly (modification): if a member changes their name, every rental row for that member must change.

2NF: CarID → CarModel (and, through it, ModelSeats) is partial, so split it out:

- **CAR** ({u}`CarID`, CarModel, ModelSeats)
- **RENTAL** ({u}`CarID`, {u}`RentalDate`, MemberID, MemberName, MilesDriven) FK (CarID) ref CAR

3NF: remove CarID → CarModel → ModelSeats and CarID, RentalDate → MemberID → MemberName:

1. **MODEL** ({u}`CarModel`, ModelSeats)
2. **CAR** ({u}`CarID`, CarModel)
   FK (CarModel) ref MODEL
3. **MEMBER** ({u}`MemberID`, MemberName)
4. **RENTAL** ({u}`CarID`, {u}`RentalDate`, MemberID, MilesDriven)
   FK (CarID) ref CAR, FK (MemberID) ref MEMBER
:::

**3. Dependencies from the rules.** In **COURSE_OFFERING** ({u}`CourseCode`, {u}`Term`, CourseTitle, InstructorID, InstructorOffice), no dependencies are given. Write the business rules you would assume, the FDs that follow from them, and the 3NF relations.

:::{note} One reasonable answer
:class: dropdown
Assume each course has one title, each offering (course in a term) has one instructor, and each instructor has one office. Then CourseCode → CourseTitle (partial), CourseCode, Term → InstructorID (full) and InstructorID → InstructorOffice (transitive). In 3NF: **COURSE** ({u}`CourseCode`, CourseTitle); **INSTRUCTOR** ({u}`InstructorID`, InstructorOffice); **COURSE_OFFERING** ({u}`CourseCode`, {u}`Term`, InstructorID) FK (CourseCode) ref COURSE, FK (InstructorID) ref INSTRUCTOR. Under different rules (for example, team-taught offerings) the answer changes, which is why you state your assumptions.
:::

**4. Anomalies in your own data.** Find a spreadsheet you use at work or in another course in which one row mixes two kinds of things (for example, orders and customers). Write one insertion, one deletion and one modification anomaly it could cause, then propose a 3NF design in the text form above.

## Key terms

normalization, redundancy, insertion anomaly, deletion anomaly, modification anomaly, functional dependency, determinant, full dependency, partial dependency, transitive dependency, valid relation, first normal form (1NF), second normal form (2NF), third normal form (3NF), Boyce-Codd and higher normal forms, denormalization

## Further reading

- For a full treatment of normalization, including Boyce-Codd and higher normal forms, see the chapter on logical database design in Hoffer, Ramesh and Topi, *Modern Database Management*.
