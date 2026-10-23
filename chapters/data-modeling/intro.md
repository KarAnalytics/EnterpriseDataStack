# Data Modeling with draw.io

An ER model is only useful if people can see it. This short, hands-on chapter surveys the tools used to draw data models and shows, step by step, how to draw a crow's foot ER diagram in **draw.io**, the free tool used in this course.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Distinguish enterprise data modeling tools from lightweight conceptual modeling tools.
2. Draw entity classes, identifiers, attributes and relationships with crow's foot symbols in draw.io.
3. Export a diagram for a report, and keep an editable copy.
:::

## Tools for data modeling

**Enterprise data modeling tools** include Microsoft Visio, Oracle SQL Developer Data Modeler, erwin Data Modeler (Quest), Idera ER/Studio and DbSchema. Commercial tools like these survive alongside free ones because they target complex enterprise needs: large-scale data governance, collaboration across teams, and automated **reverse engineering** of diagrams from existing databases (you will try Oracle's version in [Advanced SQL Queries](../sql-advanced/intro.md)).

**Lightweight conceptual modeling tools** include Lucidchart, ERDPlus, Visual Paradigm, draw.io (diagrams.net) and dbdiagram.io. Where enterprise tools focus on governance, lightweight tools are geared toward agility, real-time collaboration and the developer's experience. draw.io is open source, flexible and easy to use, and it is what this course uses.

## Drawing a crow's foot diagram in draw.io

1. Go to [www.draw.io](https://www.drawio.com) (also at diagrams.net) and choose where to save your diagrams: Google Drive, OneDrive or your device. Google Drive is convenient because the diagrams then follow you from computer to computer.
2. In the left panel, open the **Entity Relation** shape group. If you don't see it, click **+ More Shapes** at the bottom of the panel and enable it.
3. For each entity class, drag the **List** shape (second row, first column of the group) onto the canvas. Type the class name in the header and one attribute per row.
4. Drag the shape in the **first row, fifth column** into the top of the list to create the **primary key row**. Make the key bold and underlined.
5. Hover over the relationship shapes at the bottom of the group to see their names ("1 Mandatory", "0 to Many", and so on) and drag the one you need between two boxes. Connect each end to a box, then double-click the line to type the verb phrase.
6. Specify the remaining notation: `( )` for composite attributes, `{ }` for multivalued attributes and `[ ]` for derived ones; rounded corners for associative entities (Edit Style, `rounded=1`); a double border for weak entities; and a circle from the **Basic** shapes, marked D or O, for a specialization.
7. Write any assumptions you made in a text box beside the diagram.

You don't have to start from a blank canvas: every diagram in this book is a `.drawio` file you can download and open (**File > Open from > Device**, or drag the file onto the draw.io window). The animal shelter model from the previous chapter is a good one to take apart: {download}`shelter_eer.drawio <../er-model/images/shelter_eer.drawio>`.

## Exporting your model

- **Option 1: an image.** **File > Export as > PNG** (or SVG or PDF) and paste the image into your document. Keep the `.drawio` file too, so you can change the diagram later.
- **Option 2: the Word add-in.** In Microsoft Word, **Insert > Add-ins > Get Add-ins**, search for **draw.io Diagrams**, and insert diagrams directly from your draw.io folder on Google Drive or OneDrive. When the diagram changes, update it from the add-in instead of pasting a new image.

## Common mistakes

- **Lines that are not attached.** If a relationship line ends near a box instead of on it, it will not follow the box when you move it. Drag each end until the box outline is highlighted.
- **The right symbol on the wrong end.** After drawing each line, read it aloud in both directions (see [reading cardinality](../er-model/intro.md)).
- **Only exporting the image.** A PNG cannot be edited. Save the `.drawio` file with your work.

## Practice

**1. Draw the shelter.** Without looking at the shelter figure, draw the model in draw.io from the seven business rules in [Databases and the ER Model](../er-model/intro.md). Then open {download}`shelter_eer.drawio <../er-model/images/shelter_eer.drawio>` and compare. Where you differ, decide whether your version follows a different but reasonable reading of the rules (write it as an assumption) or contradicts a rule.

**2. Extend it.** Add this rule to your diagram: *the shelter has kennels (kennel number, size); an animal at the shelter occupies at most one kennel, and a kennel holds at most one animal.*

:::{note} Hint
:class: dropdown
KENNEL is a strong entity. The new relationship is one-to-one and optional on both sides: zero or one at each end.
:::

**3. Model the course.** Draw an ER diagram of a course you are taking: students, assignments, submissions and grades. Which entity is associative? Does any entity need a partial identifier (for example, if resubmissions are allowed)? Export it as a PNG and paste it into a document with your assumptions.

## Further reading

- draw.io, [Entity relationship diagrams with draw.io](https://drawio-app.com/blog/entity-relationship-diagrams-with-draw-io/) (optional).
