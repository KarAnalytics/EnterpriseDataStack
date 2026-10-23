# Databases and the ER Model

Before anyone writes `CREATE TABLE`, someone has to decide what the database is about: which things the business tracks, what it records about each one, and how those things are connected. This chapter introduces databases and database design, and then the most widely used tool for the first design step, the **entity-relationship (ER) model**, including its enhanced (EER) features. The next chapters show how to draw ER diagrams in draw.io, how to turn them into tables, and how to check those tables with normalization.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Define a database and a DBMS, name the four core operations, and list the steps of database design.
2. Identify entity classes, attributes, identifiers and relationships in a business description.
3. Read and draw minimum and maximum cardinality in crow's foot notation, and state each relationship as a business rule in both directions.
4. Decide when to use an associative entity, a weak entity or a supertype/subtype hierarchy, and classify a hierarchy as disjoint or overlapping, total or partial.
5. Read an existing ER model, such as the Northwind sample database.
:::

## Databases and DBMSs

A **database** is an organized collection of logically related data, designed to meet the information needs of an organization. It is designed, built and populated for a specific purpose: a hospital's patient database answers different questions from a retailer's sales database.

Any database must support four core operations, often abbreviated **CRUD**: create, retrieve (read), update and delete. In SQL these are `INSERT`, `SELECT`, `UPDATE` and `DELETE`.

A **database management system (DBMS)** is the software that creates and maintains databases and provides controlled access to them. Developers, business users and applications never touch the data files directly; they send requests to the DBMS, which checks permissions, enforces rules, and stores or retrieves the data. The **relational DBMS (RDBMS)**, which stores data in tables, is by far the most common kind. Oracle, Microsoft SQL Server, MySQL and PostgreSQL are all RDBMSs; this course uses Oracle in Oracle Cloud (see [Setting up Oracle Cloud](../appendix/oracle-cloud.md)).

:::{note} Think about it
A bio-pharmaceutical company has found a way to synthesize 10-HDA, a compound found in royal jelly, the "superfood" of queen bees, and hires your team as data management consultants for the research program. Before drawing anything: what makes managing this research data complex? Consider the kinds of data involved (lab experiments, samples, suppliers, patents, trials, regulators), who needs it, how it changes over time, and what must never be lost. A GenAI assistant is a good brainstorming partner for this kind of question.
:::

## Steps in database design

Database design moves from the business toward the technology:

1. **Conceptual design.** Develop a visual model of the database, the **ER model**, that captures the business rules: entities, their attributes and the relationships between them. This chapter.
2. **Logical design.** Convert the ER model into relational tables and extend it with specific information about the data: data types, indexes and constraints. See [The Relational Model](../relational-model/intro.md).
3. **Normalization.** Organize the tables to reduce redundancy and improve data integrity. See [Normalization](../normalization/intro.md).
4. **Choose a DBMS** and implement the design, for example with SQL DDL in Oracle. See [SQL: DDL, DCL and DML](../sql-ddl-dcl-dml/intro.md).

The **conceptual model** is the starting point once the requirements are known. It converts the business rules described by analysts and clients into a visual schema, and its goal is to capture as much of the *meaning* of the data as possible. A good conceptual design leads to a database that is easier to maintain and easier to extend.

## The ER model

The ER model describes a database as a collection of **entities** (things) and the **relationships** between them. It is the most popular conceptual model because:

- it is an **intuitive communication tool** that an analyst, a developer and a manager can all read;
- it is **independent of technology**: it says nothing about which DBMS you will use or how data is stored;
- it represents common **business rules** directly;
- it is a strong foundation for the rest of the development process.

### Entities

- An **entity** is a person, a place, an object, an event or a concept about which the organization wants to keep data.
- An **entity class** (or entity type) is a collection of entities that share common properties, such as CUSTOMER, PRODUCT or MOVIE. Entity classes become tables.
- An **entity instance** is a single occurrence of the class: one particular customer, one product, one movie. Instances become rows.

A common beginner's mistake is to draw an instance as a class. "Kansas City store" is an instance of STORE; it is not its own entity class.

### Attributes

An **attribute** is a named property of an entity class. Each attribute will become a column of a table. The course uses these kinds:

| Kind | Meaning | Example | Written in crow's foot as |
|---|---|---|---|
| Simple | a single value that is not broken down | an employee's salary | `Salary` |
| Composite | made of meaningful parts | an address made of street, apartment, city, state and zip | `Address (Street, City, State, Zip)` |
| Multivalued | can hold several values for one instance | an employee's phone numbers | `{Phone}` |
| Derived | can be calculated from other attributes | age, from date of birth | `[Age]` |
| Identifier (primary key) | one or more attributes that are unique for every instance | employee ID | `PK EmployeeID`, bold and underlined |

**Every entity class must have a primary key.** If you cannot find one, either the class is not really an entity class, or it is a weak entity (see below).

### Notations

Several notations exist for the same ideas: Chen, crow's foot, UML and combinations of them. Chen and crow's foot are the most popular. In **Chen** notation, entity classes are rectangles, attributes are ovals around them and relationships are diamonds. In **crow's foot** notation, each entity class is a box that lists its attributes, and relationships are lines with symbols at the ends. Chen diagrams make every concept visible, but they become crowded quickly. Crow's foot is what most design tools produce, so **this book uses crow's foot**.

:::{figure} images/attribute_notation.png
:label: fig-attribute-notation
:alt: Three crow's foot entity boxes. A regular entity class shows a PK row with an underlined primary key, then a regular attribute, a composite attribute with its parts in parentheses, a multivalued attribute in curly braces and a derived attribute in square brackets. An associative class has rounded corners and a partial identifier with a dashed underline. A weak class has a double border and a partial identifier.
How attributes and the three kinds of entity class are drawn in crow's foot notation. Open the source file in draw.io: {download}`attribute_notation.drawio <images/attribute_notation.drawio>`.
:::

## Relationships

A **relationship** is an association between instances of one or more entity classes that is of interest to the organization. It indicates that an event has occurred (a customer *placed* an order) or that there is a natural link between the classes (an employee *belongs to* a department). Relationships are labeled with verb phrases.

The **degree** of a relationship is the number of entity classes involved:

- **Unary**: an instance relates to another instance of the same class (an employee *manages* other employees).
- **Binary**: instances of two classes (a customer *places* orders). Most relationships are binary.
- **Ternary**: a simultaneous relationship among instances of three classes (a physician *prescribes* a treatment *to* a patient). In crow's foot notation a ternary relationship is drawn as an associative entity connected to all three classes.

### Cardinality

**Cardinality constraints** state how many instances of one entity class can or must be associated with each instance of another.

- The **minimum cardinality** is the lower bound. If it is zero, the relationship is **optional**; if it is one or more, it is **mandatory**.
- The **maximum cardinality** is the upper bound, typically one or many (M).

Crow's foot notation draws both at each end of the line. The symbol nearer the box is the maximum; the symbol farther from the box is the minimum.

:::{figure} images/cardinality_symbols.png
:label: fig-cardinality
:alt: Four relationship lines ending in different crow's foot symbols: two bars for mandatory one, a circle and a bar for optional one, a bar and a crow's foot for mandatory many, and a circle and a crow's foot for optional many.
The four cardinality symbols. Source: {download}`cardinality_symbols.drawio <images/cardinality_symbols.drawio>`.
:::

**Read the symbol at the far end.** To find how many PROJECTs one EMPLOYEE can be assigned to, look at the symbol next to PROJECT. Then turn every line into two sentences, one in each direction:

> Each EMPLOYEE is assigned to a minimum of **zero** and a maximum of **many** PROJECTs. Each PROJECT is assigned a minimum of **one** and a maximum of **many** EMPLOYEEs.

If a business manager would not agree with both sentences, the line is wrong. This habit catches most cardinality errors before they become database errors.

## Associative entities

When a many-to-many relationship has attributes of its own, those attributes do not belong to either side. Students enroll in many courses and each course has many students; a student's attendance and grade describe the *pair*, not the student and not the course. Model the relationship as an **associative entity class**, drawn as a box with rounded corners. It contains the attributes of the relationship and inherits the identifiers of its parent classes, which together identify its instances.

As tables, the idea is easy to see. STUDENT_PERFORMANCE needs both StudentID and CourseNo to identify a row:

| StudentID | CourseNo | Attendance | Grade |
|---|---|---|---|
| 101101 | 5011 | 95 | A- |
| 101101 | 5013 | 98 | A+ |
| 101102 | 5011 | 91 | B- |
| 101102 | 5012 | 97 | B+ |

Sometimes the inherited keys are not enough, because the same pair can occur more than once: a physician can start the same patient on a new treatment next month. Then the associative entity needs a **partial identifier** of its own (drawn with a dashed underline), such as the treatment start date, which together with the borrowed keys identifies each instance.

An associative entity is not always the right choice. If the "relationship" has its own independent identifier, such as an invoice number or a shipment ID, and other classes relate to it, it is really a **strong entity class**: draw it as an ordinary box with its own primary key.

## Weak entities

A **weak entity** cannot exist without its owner and cannot be identified by its own attributes alone. An employee's dependents mean nothing to the company without the employee, and a dependent's name alone does not identify them across all employees. Draw a weak entity with a double border and a partial identifier; its full identifier is the owner's key plus the partial identifier. The relationship to the owner is mandatory one on the owner's side.

## Enhanced ER: superclasses and subclasses

The **enhanced ER (EER)** model adds **superclass/subclass** hierarchies (also called supertype/subtype). A subclass inherits all attributes and relationships of its superclass and can have its own additional attributes and relationships. Every instance of a subclass is also an instance of its superclass.

### When to use subclasses

Use subclasses only in these three cases:

1. **The subclasses have different attributes** (the most common case). Permanent employees have a salary and health coverage; contract employees have an hourly rate.
2. **A relationship applies to some subclasses only.** If only contract employees are placed by staffing agencies, draw that relationship to the CONTRACT subclass. A relationship drawn to the superclass means that *every* subclass participates in it.
3. **The cardinality of a relationship differs across subclasses** (rare).

If none of the three applies, do not draw subclasses; add a type attribute (such as `EmployeeType`) to the entity class instead.

### Disjoint or overlapping, total or partial

Two constraints describe every hierarchy:

- **Disjoint (d)**: an instance belongs to at most one subclass. **Overlapping (o)**: an instance can belong to more than one. The letter goes in the circle.
- **Total specialization**: every instance must belong to some subclass (double line from the superclass to the circle). **Partial specialization**: an instance does not need to belong to any subclass (single line).

Together they say how many subclasses each superclass instance belongs to:

| Combination | Each superclass instance belongs to |
|---|---|
| Disjoint and partial | zero or one subclass |
| Disjoint and total | exactly one subclass |
| Overlapping and partial | zero, one or many subclasses |
| Overlapping and total | one or many subclasses |

Phrases in a business description usually tell you which to use. "Every account is either a checking or a savings account" means disjoint and total. "A person may be both an employee and a student" means overlapping. "Some animals are neither" means partial.

## Worked example: an animal shelter

A regional animal shelter wants a database. After interviews, the analyst wrote these business rules:

1. The shelter records each **animal** it takes in: an animal ID, name, species, date of birth (estimated if unknown) and intake date. Staff often want each animal's age.
2. For **dogs** the shelter also records breed and size. For **cats** it records whether the cat is indoor-only and whether it has been tested for FIV. Rabbits, birds and other animals have no extra information.
3. Every animal can have any number of **vet visits**. Visits are numbered 1, 2, 3, ... separately for each animal, and each visit has a date, a reason and a cost.
4. **Volunteers** (ID, name, email, start date) may foster animals at home. A volunteer can foster many animals over time and an animal can be fostered by several volunteers. For each foster stay the shelter records the start and end dates. The same volunteer may foster the same animal again later.
5. An experienced volunteer may mentor new volunteers. Each volunteer has at most one mentor.
6. **Adopters** have an ID, name, address and one or more phone numbers. An adopter is recorded only when they adopt.
7. Each **adoption** gets its own adoption ID, a date and a fee. An adoption places one animal with one adopter. Animals that are returned can be adopted again, and a person may adopt several animals.

:::{figure} images/shelter_eer.png
:label: fig-shelter
:alt: EER diagram of the animal shelter. VOLUNTEER connects to the associative entity FOSTER_STAY, which connects to ANIMAL. VOLUNTEER has a unary mentors relationship. ANIMAL connects to the weak entity VET_VISIT and to the strong entity ADOPTION, which connects to ADOPTER. ANIMAL has a disjoint, partial specialization into DOG and CAT.
An EER model of the shelter. Download it and open it in draw.io to explore: {download}`shelter_eer.drawio <images/shelter_eer.drawio>`.
:::

How each rule became part of {numref}`fig-shelter`:

- **Rule 1.** ANIMAL is a strong entity with identifier AnimalID. Age changes every day, so it is the derived attribute `[Age]`, calculated from DateOfBirth.
- **Rule 2.** DOG and CAT have different attributes, so subclasses are justified (case 1). The circle shows **D** because an animal cannot be both, and the line from ANIMAL is single (**partial**) because rabbits and birds belong to neither subclass. Species stays on ANIMAL as an ordinary attribute.
- **Rule 3.** A vet visit cannot exist without its animal, and VisitNo restarts for every animal, so VET_VISIT is a **weak entity** with partial identifier VisitNo. Each ANIMAL receives zero or many VET_VISITs; each VET_VISIT belongs to exactly one ANIMAL.
- **Rule 4.** Volunteers and animals are many-to-many, and the dates describe the pair, so FOSTER_STAY is an **associative entity**. Because the same volunteer can foster the same animal twice, the inherited keys are not unique on their own; StartDate is added as a **partial identifier**.
- **Rule 5.** A **unary** relationship on VOLUNTEER: each volunteer is mentored by zero or one volunteer, and mentors zero or many.
- **Rule 6.** `Address` is composite and `{Phone}` is multivalued.
- **Rule 7.** ADOPTION has its own identifier, AdoptionID, so it is a **strong entity**, not an associative one. Each ANIMAL is placed in zero or many ADOPTIONs (zero while it is still at the shelter, more than one if it is returned); each ADOPTER makes one or many ADOPTIONs, because adopters are recorded only when they adopt.

## Reading an existing model: Northwind

Much of the time you will read ER models rather than draw them. **Northwind** is a sample database created by Microsoft for an imaginary food trading company, and it is used in many database courses and tutorials (this book uses it again in the [ETL chapter](../etl-northwind/intro.md)). Its ER model, available in many places online, has 13 entity classes: Suppliers, Products, Categories, Orders, Order Details, Customers, Customer_Customer_Demo, Customer_Demographics, Employees, Employee_Territories, Territories, Region and Shippers.

Reading the model tells you what the business does:

- Northwind manages orders for a set of products; it deals with suppliers, customers and shippers, and keeps employee information.
- Relationships are verbs: *suppliers supply products*, *employees complete orders*, *customers place orders*, *shippers ship orders*. Many published diagrams, including Northwind's, leave the verb phrases off the lines; you supply them as you read.
- ORDER_DETAILS is an associative entity between ORDERS and PRODUCTS, with its own attributes (unit price, quantity, discount).
- EMPLOYEES has a unary relationship (*reports to*) that records each employee's supervisor.
- Each entity class becomes a table. There can be additional tables for multivalued attributes and many-to-many relationships, such as EMPLOYEE_TERRITORIES.

## Common mistakes

- **Reading cardinality at the near end.** The symbol next to a box describes how many of *that* box relate to one instance of the *other* box.
- **Forgetting minimums.** "Many" is not enough. Decide whether zero is allowed, and check it against the business rules (can a customer exist with no orders yet?).
- **Associative entity by reflex.** If it has its own ID or its own relationships, it is a strong entity.
- **Subclasses without a reason.** Different names are not a reason; different attributes, relationships or cardinalities are.
- **Unstated assumptions.** When the description is silent, choose a reasonable rule and write it down as an assumption next to the diagram.

:::{warning} Check the AI
Ask a GenAI assistant to "draw an ER diagram for an animal shelter" and you will get something that looks professional. Before you trust it, test every line against the business rules, in both directions. Typical problems:

- **ANIMAL to ADOPTION drawn as mandatory one-to-one.** That says every animal has exactly one adoption, but animals still at the shelter have none, and returned animals are adopted again.
- **FOSTER_STAY identified by (VolunteerID, AnimalID) only.** This breaks the first time a volunteer fosters the same animal twice (rule 4).
- **A subclass for every species,** each with no attributes of its own. None of the three cases for subclasses applies.
- **A missing identifier** on an entity class, or foreign key columns listed inside the boxes of a conceptual diagram.

The assistant does not know your business rules unless you give them, and it will not tell you when it has guessed. You are the one who has to say whether a manager would agree with each line.
:::

## Practice

**1. Reading cardinality.** In {numref}`fig-shelter`, write the two sentences for the line between ADOPTION and ADOPTER.

:::{note} Answer
:class: dropdown
Each ADOPTION is made by **exactly one** ADOPTER (the mandatory-one symbol next to ADOPTER). Each ADOPTER makes **one or many** ADOPTIONs (the mandatory-many symbol next to ADOPTION). The minimum is one because adopters are recorded only when they adopt (rule 6).
:::

**2. Disjoint or overlapping, total or partial?** Classify each hierarchy. (a) Vehicles in a dealership's inventory are new or used. (b) Members of a museum are students, seniors, both, or neither (for discounts). (c) A hospital's staff are physicians, nurses or administrators; nobody holds two roles, and every staff member has one of them. (d) A bookstore's products are books or gift items; some products, such as gift cards, are neither.

:::{note} Answer
:class: dropdown
(a) Disjoint, total. (b) Overlapping, partial. (c) Disjoint, total. (d) Disjoint, partial.
:::

**3. Associative or strong?** A climbing gym records which members attend which classes, with the check-in time of each attendance. Later, the gym gives each check-in a scan number and records which front-desk employee scanned it. How should ATTENDANCE be modeled before and after the change?

:::{note} Answer
:class: dropdown
Before: an associative entity between MEMBER and CLASS (many-to-many with an attribute, CheckInTime). After: ATTENDANCE has its own identifier (ScanNo) and its own relationship to EMPLOYEE, so it becomes a strong entity class with one-to-many relationships to MEMBER, CLASS and EMPLOYEE.
:::

**4. Subclass or attribute?** A parking garage records vehicles (plate, make, color). Cars and motorcycles are recorded the same way. Electric vehicles also have a charging-port type, and only electric vehicles have charging sessions. What should be a subclass?

:::{note} Answer
:class: dropdown
ELECTRIC_VEHICLE is a subclass of VEHICLE: it has its own attribute (case 1) and its own relationship to CHARGING_SESSION (case 2). Car versus motorcycle is a `VehicleType` attribute, because neither has its own attributes or relationships. The specialization is partial, because most vehicles are not electric.
:::

**5. Read a model.** Find a published ER diagram of the Northwind database. List three relationships as verb-phrase sentences in both directions, including their minimum and maximum cardinalities.

## Key terms

database, DBMS, RDBMS, CRUD, conceptual design, logical design, ER model, entity, entity class, entity instance, attribute (simple, composite, multivalued, derived), identifier, relationship, degree (unary, binary, ternary), minimum and maximum cardinality, optional, mandatory, crow's foot notation, Chen notation, associative entity, partial identifier, weak entity, EER, superclass/subclass, disjoint, overlapping, total and partial specialization

## Further reading

- For complete coverage of ER and EER modeling, see the chapters on modeling data in the organization and on the enhanced E-R model in Hoffer, Ramesh and Topi, *Modern Database Management*.
