# The Relational Model

An ER model cannot be implemented in a DBMS directly: Oracle does not know what a crow's foot is. **Logical design** translates the ER model into a **relational model** of tables, columns and keys that a relational DBMS can build. This chapter explains the parts of a relational model and gives a step-by-step procedure for mapping every ER and EER construct from [Databases and the ER Model](../er-model/intro.md) to tables.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Name the three components of a relational model and the role of primary and foreign keys.
2. Write a relational model in text form, and read its graphical form.
3. Map strong entities, composite and multivalued attributes, one-to-many, many-to-many, unary and ternary relationships, associative entities, weak entities and subclasses to relations.
4. Apply the seven mapping steps to a complete EER model.
:::

## From diagram to tables

Many published diagrams, including the Northwind model, take a **hybrid approach**: crow's foot boxes that also list the foreign key columns, mixing the conceptual and logical designs. It saves time, but it hides decisions, and the exact translation from diagram to code can still be confusing. This chapter keeps the two steps separate: first the ER model (business rules), then the relational model (tables and keys).

**Logical design** maps the entities, relationships and multivalued attributes of the ER model into a logical schema described by a **relational model**, which has three components:

1. **Data structure**: tables (relations) made of rows and columns. Each row describes one instance; each cell holds one value.
2. **Data manipulation**: SQL operations for retrieving and modifying the data.
3. **Data integrity**: mechanisms that implement business rules and keep the data correct as it changes.

The basic idea of the translation:

- create a relation (table) for each entity class;
- for each relationship, either create a separate table or, where permitted, add a foreign key to the "many" side;
- create tables for multivalued attributes;
- add foreign keys where necessary;
- think about the data type of each attribute, ready for implementation with SQL DDL.

## Keys

Keys are special fields of two types:

- A **primary key** uniquely identifies each row of a relation, such as an employee number. It guarantees that all rows are unique, and it can never be empty (null).
- A **foreign key** is an attribute that lets a dependent relation refer to its parent relation through a common attribute: `ORDERS.CustomerID` refers to `CUSTOMERS.CustomerID`. Foreign keys are what link tables together, an integral property of an RDBMS.

Keys can be **simple** (a single column) or **composite** (more than one column), and a DBMS usually indexes them to speed up queries.

The DBMS uses keys to enforce two integrity rules. **Entity integrity**: primary key values are unique and never null. **Referential integrity**: every foreign key value matches an existing primary key value in the parent table (or is null, if that is allowed). Try to insert an order for a customer who does not exist, and the DBMS rejects it.

## Two forms of a relational model

The **text-based form** lists each relation with its columns. Primary key columns are underlined, and foreign keys are listed below the relation:

> **ADOPTION** ({u}`AdoptionID`, AdoptionDate, Fee, AnimalID, AdopterID)
> FK (AnimalID) ref ANIMAL, FK (AdopterID) ref ADOPTER

The **graphical form** draws each relation as a box of columns, with an arrow from each foreign key to the primary key it references. It is more expressive, but the text form is simpler and less error-prone, so this book uses it for most examples.

:::{figure} images/shelter_relational.png
:label: fig-shelter-relational
:alt: Four tables drawn as boxes: ADOPTER, ADOPTION, ANIMAL and VET_VISIT. Arrows run from ADOPTION to ADOPTER and to ANIMAL, and from VET_VISIT to ANIMAL, showing the foreign key references. VET_VISIT has a composite primary key of AnimalID and VisitNo.
Part of the animal shelter's relational model in graphical form. Arrows point from a foreign key to the table it references. In VET_VISIT, AnimalID is both part of the primary key and a foreign key. Source: {download}`shelter_relational.drawio <images/shelter_relational.drawio>`.
:::

## Mapping each construct

### A single entity class

Each entity class becomes a relation; each simple attribute becomes a column; each instance becomes a row.

> **CUSTOMER** ({u}`CustomerID`, CustomerName, CustomerAddress, CustomerPostalCode)

| CustomerID | CustomerName | CustomerAddress | CustomerPostalCode |
|---|---|---|---|
| 101002 | Avery Lin | 125 Overhill Dr, Lawrence | 66045 |
| 101022 | Jordan Patel | 121 Helen St, Topeka | 66603 |

### Composite attributes

Each component of a composite attribute becomes its own column:

> **CUSTOMER** ({u}`CustomerID`, CustomerName, CustomerStreet, CustomerCity, CustomerState, CustomerPostalCode)

### Multivalued attributes

A relation cannot hold several values in one cell, so a multivalued attribute moves to a separate relation, together with the owner's key. Both columns form a composite primary key:

> **EMPLOYEE** ({u}`EmployeeID`, EmployeeName, EmployeeAddress)
> **EMPLOYEE_SKILL** ({u}`EmployeeID`, {u}`Skill`)
> FK (EmployeeID) ref EMPLOYEE

### One-to-many relationships

There are two options.

**Option 1: add a foreign key to the "many" side.** Each order belongs to one customer, so ORDER carries the customer's key:

> **CUSTOMER** ({u}`CustomerID`, CustomerName, CustomerAddress)
> **ORDER** ({u}`OrderID`, OrderDate, CustomerID)
> FK (CustomerID) ref CUSTOMER

**Option 2: create a mapping relation** that pairs the two keys:

> **CUSTOMER** ({u}`CustomerID`, CustomerName, CustomerAddress)
> **ORDER** ({u}`OrderID`, OrderDate)
> **CUSTOMER_ORDERS** ({u}`CustomerID`, {u}`OrderID`)
> FK (CustomerID) ref CUSTOMER, FK (OrderID) ref ORDER

Option 1 uses fewer tables and fewer joins and is the usual choice. Option 2 keeps the ORDER table free of customer information, and it is easier to change later if the relationship becomes many-to-many. Both are correct.

### Many-to-many relationships

A many-to-many relationship always becomes its own relation, whose primary key combines the keys of both sides:

> **EMPLOYEE** ({u}`EmployeeID`, EmployeeName)
> **PROJECT** ({u}`ProjectID`, ProjectDomain, ProjectBudget)
> **EMPLOYEE_DEPLOY** ({u}`EmployeeID`, {u}`ProjectID`)
> FK (EmployeeID) ref EMPLOYEE, FK (ProjectID) ref PROJECT

### Unary relationships

A **one-to-many** unary relationship becomes a foreign key that refers to the same table. Each employee has at most one manager, who is also an employee:

> **EMPLOYEE** ({u}`EmployeeID`, EmployeeName, EmployeeDoB, ManagerID)
> FK (ManagerID) ref EMPLOYEE (EmployeeID)

A **many-to-many** unary relationship (an employee can report to several managers) needs a separate relation with two foreign keys to the same table:

> **EMPLOYEE** ({u}`EmployeeID`, EmployeeName)
> **REPORTING_MAP** ({u}`SubordinateID`, {u}`ManagerID`)
> FK (SubordinateID) ref EMPLOYEE (EmployeeID), FK (ManagerID) ref EMPLOYEE (EmployeeID)

### Ternary relationships

A ternary relationship, drawn as an associative entity, becomes a relation whose key combines the keys of all three classes, plus its own attributes:

> **PATIENT_TREATMENT** ({u}`PatientID`, {u}`PhysicianID`, {u}`TreatmentCode`, TreatmentDate, TreatmentTime, TreatmentResults)
> FK (PatientID) ref PATIENT, FK (PhysicianID) ref PHYSICIAN, FK (TreatmentCode) ref TREATMENT

### Associative entities

An associative entity becomes a relation whose composite primary key combines the keys of the classes it connects, plus any partial identifier:

> **STUDENTS** ({u}`StudentID`, StudentName)
> **CLUBS** ({u}`ClubID`, ClubAddress)
> **LEADERSHIP_ROLES** ({u}`StudentID`, {u}`ClubID`, {u}`RoleDescription`)
> FK (StudentID) ref STUDENTS, FK (ClubID) ref CLUBS

Here RoleDescription is a partial identifier: one student can hold two different roles in the same club.

If the "associative" entity has its own identifier, it is really a **strong entity**, and the keys of the related classes are ordinary foreign keys:

> **SHIPMENT** ({u}`ShipmentID`, ShipmentDate, ShipmentAmount, CustomerID, VendorID)
> FK (CustomerID) ref CUSTOMER, FK (VendorID) ref VENDOR

### Weak entities

The primary key of the owner (strong) entity is added to the weak entity and becomes part of its primary key:

> **EMPLOYEE** ({u}`EmployeeID`, EmployeeName)
> **DEPENDENT** ({u}`EmployeeID`, {u}`FirstName`, {u}`MiddleInitial`, {u}`LastName`, DateOfBirth, Gender)
> FK (EmployeeID) ref EMPLOYEE

### Superclasses and subclasses

Each subclass becomes a relation with the **same primary key as the superclass**, which is also a foreign key to the superclass:

> **STUDENT** ({u}`StudentID`, StudentName)
> **UNDERGRADUATE** ({u}`StudentID`, UndergraduateYear)
> FK (StudentID) ref STUDENT
> **GRADUATE** ({u}`StudentID`, GraduateYear, ThesisCompletion)
> FK (StudentID) ref STUDENT

### Derived attributes

Derived attributes are usually not stored, because they can be calculated when needed, and a stored age is wrong a day later. Calculate them in queries or views instead.

## The seven mapping steps

1. Create relations for all **strong entity classes** (splitting composite attributes into their parts).
2. Create relations for **multivalued attributes**, if any.
3. Create relations for **weak entity classes**, if any.
4. Create relations for **associative entity classes**, if any.
5. Create relations for **many-to-many relationships**, if any.
6. For each **one-to-many relationship**, add a foreign key to the "many" side or create a mapping relation.
7. Create relations for **subclasses**, if any. A subclass has the same primary key as its superclass, with a foreign key to the superclass.

## Worked example: the animal shelter

Applying the steps to the shelter's EER model ({numref}`fig-shelter` in [Databases and the ER Model](../er-model/intro.md)) gives nine relations:

1. **VOLUNTEER** ({u}`VolunteerID`, Name, Email, StartDate, MentorID)
   FK (MentorID) ref VOLUNTEER (VolunteerID)
2. **ANIMAL** ({u}`AnimalID`, Name, Species, DateOfBirth, IntakeDate)
3. **ADOPTER** ({u}`AdopterID`, Name, Street, City, State, Zip)
4. **ADOPTER_PHONE** ({u}`AdopterID`, {u}`Phone`)
   FK (AdopterID) ref ADOPTER
5. **VET_VISIT** ({u}`AnimalID`, {u}`VisitNo`, VisitDate, Reason, Cost)
   FK (AnimalID) ref ANIMAL
6. **FOSTER_STAY** ({u}`VolunteerID`, {u}`AnimalID`, {u}`StartDate`, EndDate)
   FK (VolunteerID) ref VOLUNTEER, FK (AnimalID) ref ANIMAL
7. **ADOPTION** ({u}`AdoptionID`, AdoptionDate, Fee, AnimalID, AdopterID)
   FK (AnimalID) ref ANIMAL, FK (AdopterID) ref ADOPTER
8. **DOG** ({u}`AnimalID`, Breed, Size)
   FK (AnimalID) ref ANIMAL
9. **CAT** ({u}`AnimalID`, IndoorOnly, FIVTested)
   FK (AnimalID) ref ANIMAL

Which step produced each relation? Steps 1 (VOLUNTEER, ANIMAL, ADOPTER, ADOPTION), 2 (ADOPTER_PHONE), 3 (VET_VISIT), 4 (FOSTER_STAY), 6 (MentorID in VOLUNTEER; AnimalID and AdopterID in ADOPTION) and 7 (DOG, CAT). The model has no many-to-many relationship without attributes, so step 5 adds nothing. `[Age]` and `[DaysInFoster]` are derived and not stored.

Notice that ADOPTION has a single-column key because it is a strong entity. Had it been associative, its key would have been (AnimalID, AdopterID), and a returned animal could never be adopted again by the same person.

## Common mistakes

- **Foreign key on the "one" side.** In a one-to-many relationship, the key of the "one" side goes into the "many" side, never the reverse: a customer row cannot hold a list of order IDs.
- **A multivalued attribute left in the table** as a comma-separated list. It needs its own relation.
- **Forgetting the owner's key** in a weak entity or a subclass relation.
- **A missing FK line.** Every foreign key needs its `FK (...) ref TABLE` line; otherwise the DBMS will not enforce it.
- **Storing derived attributes** that will go out of date.

:::{warning} Check the AI
GenAI assistants convert diagrams to tables quickly, and often confidently get the keys wrong. When you ask one to map a model, check three things: that every weak entity and associative entity has a *composite* key that includes the borrowed keys (assistants often invent a surrogate ID instead, which silently drops the rule that makes the entity weak); that each one-to-many foreign key sits on the "many" side; and that every multivalued attribute has its own relation rather than a comma-separated column. If the assistant adds tables or columns that are not in your model, ask it which business rule they come from.
:::

## Practice

**1. Map a model.** TRAINER (PK TrainerID, Name) leads zero or many SESSIONs; each SESSION (PK SessionID, SessionDate, Room) is led by exactly one TRAINER. MEMBER (PK MemberID, Name, {FitnessGoal}) books zero or many SESSIONs, and each SESSION has one or many MEMBERs; for each booking the gym records the booking time. Write the relational model.

:::{note} Answer
:class: dropdown
1. **TRAINER** ({u}`TrainerID`, Name)
2. **SESSION** ({u}`SessionID`, SessionDate, Room, TrainerID)
   FK (TrainerID) ref TRAINER
3. **MEMBER** ({u}`MemberID`, Name)
4. **MEMBER_GOAL** ({u}`MemberID`, {u}`FitnessGoal`)
   FK (MemberID) ref MEMBER
5. **BOOKING** ({u}`MemberID`, {u}`SessionID`, BookingTime)
   FK (MemberID) ref MEMBER, FK (SessionID) ref SESSION
:::

**2. One-to-many, two ways.** A library BRANCH (BranchID, BranchName) has many LAPTOPs (LaptopTag, Model) for loan; each laptop belongs to exactly one branch. Write both options for this relationship.

:::{note} Answer
:class: dropdown
Option 1: **BRANCH** ({u}`BranchID`, BranchName); **LAPTOP** ({u}`LaptopTag`, Model, BranchID) FK (BranchID) ref BRANCH.

Option 2: **BRANCH** ({u}`BranchID`, BranchName); **LAPTOP** ({u}`LaptopTag`, Model); **BRANCH_LAPTOP** ({u}`BranchID`, {u}`LaptopTag`) FK (BranchID) ref BRANCH, FK (LaptopTag) ref LAPTOP.

In option 2, making LaptopTag alone the primary key of BRANCH_LAPTOP would enforce the rule that a laptop belongs to only one branch.
:::

**3. Unary and EER.** A company's employees (EmployeeID, Name) mentor other employees; an employee can have several mentors and mentor several people. Employees are either SALARIED (AnnualSalary) or HOURLY (HourlyRate), never both. Write the relations.

:::{note} Answer
:class: dropdown
1. **EMPLOYEE** ({u}`EmployeeID`, Name)
2. **MENTORING** ({u}`MenteeID`, {u}`MentorID`)
   FK (MenteeID) ref EMPLOYEE (EmployeeID), FK (MentorID) ref EMPLOYEE (EmployeeID)
3. **SALARIED** ({u}`EmployeeID`, AnnualSalary)
   FK (EmployeeID) ref EMPLOYEE
4. **HOURLY** ({u}`EmployeeID`, HourlyRate)
   FK (EmployeeID) ref EMPLOYEE
:::

**4. Your own diagram.** Take the course model you drew in [Data Modeling with draw.io](../data-modeling/intro.md) (practice 3) and map it with the seven steps. Label each relation with the step that produced it.

## Key terms

logical design, relational model, relation, data structure, data manipulation, data integrity, primary key, foreign key, simple key, composite key, entity integrity, referential integrity, text-based and graphical relational models, mapping relation, unary relationship mapping, ternary relationship mapping

## Further reading

- For a complete treatment of logical design, see the chapter on logical database design and the relational model in Hoffer, Ramesh and Topi, *Modern Database Management*.
