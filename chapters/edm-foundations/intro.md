# EDM Foundations

Every report, dashboard, forecast and AI model in an organization rests on data that someone collected, cleaned, stored, organized and protected. **Enterprise data management (EDM)** is the discipline that does that work. This chapter explains why organizations manage data at all (to make decisions), how raw data becomes knowledge, and what the parts of an EDM program are. It ends with the command-line skills you will use throughout the book.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Distinguish operational, tactical and strategic decisions and the data each needs.
2. Explain the data journey from data to information, knowledge and wisdom.
3. Name the seven components of enterprise data management and describe each one.
4. Navigate a file system and create, edit, copy, move and delete files from the command line.
:::

## Decision-making levels

Organizations use data to make decisions at three levels:

| Level | Who decides | Time frame | Example |
|---|---|---|---|
| **Operational** | front-line staff and supervisors | day to day, short term | Approve this loan application; restock aisle 7 today. |
| **Tactical** | middle management | weeks to months, medium term | Which stores need an extra cashier on weekends next quarter? |
| **Strategic** | senior management | years, long term | Should we open stores in a new region? |


:::{figure} images/decision_levels.png
:label: fig-decision-levels
:alt: A three-level pyramid: operational decisions at the base, tactical in the middle, strategic at the top, each with an example.
The three levels of decision making, from day-to-day operations to long-term strategy.
:::

Operational decisions run on **online transaction processing (OLTP)** systems: the databases behind point-of-sale terminals, banking apps and order forms, which record each transaction as it happens. Tactical and strategic decisions need data that has been integrated across those systems and summarized over time, which is the job of **data warehouses** (Part IV of this book). A good EDM program serves all three levels from the same, consistent data.

## The data journey

Data becomes useful in stages, often drawn as a pyramid known as **DIKW** (data, information, knowledge, wisdom). Consider a grocery chain:

| Stage | What it adds | Example |
|---|---|---|
| **Data** | raw facts, no context | `112, 40217, 3, 2026-03-14 09:12` |
| **Information** | meaning: what the facts describe | Store 112 sold 3 umbrellas (product 40217) on Saturday, March 14 at 9:12 a.m. |
| **Knowledge** | context: patterns across many facts | Umbrella sales at store 112 triple on rainy weekends. |
| **Wisdom** | applied: a decision or action | Send extra umbrellas to store 112 whenever weekend rain is forecast. |


:::{figure} images/dikw.png
:label: fig-dikw
:alt: A four-level pyramid: data at the base, then information, knowledge and wisdom, each with the grocery example.
The data journey: each level adds meaning, context and finally action.
:::

Each step up the pyramid needs more data management: the raw transaction must be recorded accurately (database), linked to product and store details (integration), cleaned (wrangling), summarized over months and joined to weather data (warehousing), and handled under the company's policies (governance).

## Enterprise data management

**Enterprise data management** is the proper management of an organization's data by establishing the processes that convert raw data into knowledge. It has seven components:


:::{figure} images/edm_components.png
:label: fig-edm-components
:alt: A wheel of seven segments around a hub labeled EDM: planning, information extraction, data integration, data wrangling, database management, data warehousing and data governance, with arrows showing a continuous cycle.
The seven components of enterprise data management form a continuous cycle.
:::

1. **Planning.** Define the goals, scope and requirements of the EDM ecosystem: the business objectives, the data sources, the resources needed, and the costs and benefits of the alternatives.
2. **Information extraction** (also called **ingestion**). Acquire raw data from its sources, which can be anything from internal databases and surveys to websites, Internet-of-Things sensors and system logs.
3. **Data integration.** Combine data from different sources into a single, unified view. Integration is often called **extract, transform, load (ETL)**, although strictly ETL is one way of doing integration (Part VI).
4. **Data wrangling** (also called **quality control**). Make the data accurate and reliable through **data profiling** (examining the data and summarizing what it contains) and **data processing** (adjusting the data so it is organized, readable and ready for use). Part II.
5. **Database management.** Systems designed for the CRUD operations (create, read, update, delete) that support transaction processing (OLTP). The relational DBMS is the most common kind, and SQL was developed for it. Parts III and V.
6. **Data warehousing** (closely tied to **business intelligence**). Systems designed for efficient querying and reporting, supporting **online analytical processing (OLAP)**, and the data source for machine learning and other modeling. Big data platforms, graph databases and vector databases can be seen as advanced extensions of this layer. Parts IV, VII and VIII.
7. **Data governance.** The policies, standards and controls that ensure data is handled securely, ethically and consistently from beginning to end. Governance is often treated as routine, but it is critical: it applies to every other component and to every decision-making level, and it keeps evolving with technology and law. Part II.

## Beyond the core components

Several newer areas build on these foundations:

- **Big data**: storing and processing data too large for a single server, with platforms such as Hadoop and Spark (Part VIII).
- **Graph data management**: databases that store highly connected data as nodes and relationships with properties, and the basis of **knowledge graphs** (Part VII).
- **Modern architectures**: designs that combine on-demand cloud services with established systems, using ideas such as the **data mesh**, the **data fabric** and the **lakehouse**.
- **Vector databases**: stores for **vector embeddings**, high-dimensional numerical representations of unstructured data such as text, images or audio. Generative AI applications and large language models use them to find relevant content quickly.

:::{note} Think about it
Your project team needs a shared table of basic information about its members: name, work experience, program, education, interests and availability. Walk through the seven EDM components for this small task. What is the plan? Where does the data come from? Where will it be stored and who can edit it? What should happen if a member prefers to leave a cell blank? Even a one-table project touches most of the components.
:::

## Using the terminal

The **terminal** (or **command-line interface**, CLI) is a text-based program for controlling a computer: instead of clicking icons, you type commands. A graphical interface is like ordering from a menu with pictures, friendly but limited to what is shown. The command line is like talking to the chef: you can make very specific requests, combine steps, and automate them. Many data platforms, including the Hadoop and Spark clusters in Part VIII, are used mainly through a Linux command line, and command-line skills do not go out of date.

macOS and Linux include a terminal. On Windows, Command Prompt and PowerShell use different commands; for the commands below, use **Git Bash** or the **Windows Subsystem for Linux**, or practice on a free online Linux machine (see the practice section).

### Navigating

Your file system is a tree of folders (directories).

| Command | What it does |
|---|---|
| `pwd` | print working directory: shows where you are |
| `ls` | list the files and folders here (`ls -l` adds sizes and dates) |
| `cd folder` | change directory into `folder`; `cd ..` goes up one level, `cd ~` goes home |

### Working with files and folders

| Command | What it does |
|---|---|
| `mkdir name` | make a new directory |
| `touch file.txt` | create an empty file |
| `cp a.txt b.txt` | copy a file |
| `mv a.txt reports/` | move (or rename) a file |
| `cat file.txt` | display a file's contents |
| `rm file.txt` | remove a file. **There is no undo and no recycle bin.** |
| `rm -r folder` | remove a folder and everything in it |

### Editing a file with vi

`vi` (visual editor) is available on almost every Linux system:

1. Open the file: `vi notes.txt`.
2. Press **i** to enter insert mode, then type.
3. Press **Esc** to leave insert mode.
4. Type `:wq` and press **Enter** to save (`w` for write) and quit (`q`). (`:q!` quits without saving.)

### A short session

```bash
pwd                      # /home/student
mkdir edm                # create a project folder
cd edm
touch members.csv        # an empty file
vi members.csv           # add a header line, then :wq
cat members.csv          # check the contents
cp members.csv backup.csv
ls -l                    # two files
rm backup.csv            # gone for good
cd ..
```

## Common mistakes

- **Treating integration and ETL as synonyms.** ETL is one approach to integration; data can also be integrated by replication, virtualization or streaming.
- **Skipping governance until the end.** Privacy and security decisions made after the database exists are expensive to retrofit.
- **Running `rm -r` in the wrong folder.** Run `pwd` first, and list what you are about to delete with `ls`.
- **Confusing information with knowledge.** A report of last month's sales is information; understanding *why* sales moved is knowledge.

:::{warning} Check the AI
GenAI assistants are very good at producing terminal commands, and that is exactly why you should read every command before running it. An assistant asked to "clean up my project folder" may suggest `rm -rf *` or a `find ... -delete` that removes more than you intended, and it cannot see which folder you are in. Before you paste a command: ask the assistant to explain each part and flag, run `pwd` and `ls` to confirm where you are, and try anything destructive on a copy first.
:::

## Practice

**1. Decision levels.** Classify each decision as operational, tactical or strategic: (a) a hospital decides whether to build a new outpatient center; (b) a pharmacist checks a prescription against a patient's allergies; (c) a regional manager sets next quarter's nurse staffing levels by shift.

:::{note} Answer
:class: dropdown
(a) Strategic. (b) Operational. (c) Tactical.
:::

**2. The data journey.** A bike-share company records `dock 31, bike 1207, 2026-06-02 17:45, status=empty`. Write one example each of information, knowledge and wisdom that could follow from data like this.

:::{note} One possible answer
:class: dropdown
Information: dock 31 near the stadium was empty at 5:45 p.m. on June 2. Knowledge: docks near the stadium empty out within an hour before evening events. Wisdom: move bikes to the stadium docks two hours before every scheduled event.
:::

**3. EDM components.** Which EDM component does each activity belong to? (a) Merging customer lists from the web store and the physical stores into one list. (b) Deciding which customer data may be shared with a marketing partner. (c) Finding that 8% of phone numbers have only nine digits. (d) Pulling posts from a social media API every hour.

:::{note} Answer
:class: dropdown
(a) Data integration. (b) Data governance. (c) Data wrangling (profiling). (d) Information extraction (ingestion).
:::

**4. Terminal.** Using the free [Linux container sandbox](https://linuxcontainers.org/incus/try-it/) (about 30 minutes of access) or your own terminal: create a folder `edm`, create `todo.txt` inside it with two lines of text using `vi`, copy it to `todo_backup.txt`, rename the backup to `old_todo.txt`, display it with `cat`, and finally delete the whole `edm` folder. Then work through the first two modules of the interactive [LinuxSurvival](https://linuxsurvival.com) tutorial.

:::{note} Answer
:class: dropdown
```bash
mkdir edm && cd edm
vi todo.txt              # i, type two lines, Esc, :wq
cp todo.txt todo_backup.txt
mv todo_backup.txt old_todo.txt
cat old_todo.txt
cd ..
rm -r edm
```
:::

## Key terms

operational, tactical and strategic decisions, OLTP, OLAP, DIKW (data, information, knowledge, wisdom), enterprise data management, planning, information extraction (ingestion), data integration, ETL, data wrangling, data profiling, data processing, database management, CRUD, data warehousing, business intelligence, data governance, big data, graph data management, data mesh, data fabric, lakehouse, vector database, terminal (CLI)

## Further reading

- DAMA International, *DAMA-DMBOK: Data Management Body of Knowledge* (2nd ed.), for the professional framework of data management functions.
