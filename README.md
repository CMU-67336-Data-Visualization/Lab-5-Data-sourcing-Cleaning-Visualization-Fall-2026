# Lab 5: Data Sourcing, Cleaning, and Visualization

67-336 Data Visualization, Fall 2026

## Overview

This lab goes with the AI and Data Cleaning lecture. You will work through the same steps a data analyst follows on the job: find a dataset, check its quality, clean it, and choose the chart that fits what the data actually says.

By the end you will have a cleaned dataset and two charts published on a live website. You should also be able to explain why you made each choice, not just how.

## Learning objectives

By the end of this lab, you will be able to:

1. Find and evaluate real world datasets from reliable sources
2. Spot common data quality problems and fix them
3. Choose a chart type on purpose, based on the question you are asking
4. Build charts in Observable Framework from cleaned data
5. Explain how a chart made from messy data differs from one made from clean data
6. Publish your finished page on Vercel

## Before you start

You should have finished Lab 0 (Git and Node setup), Lab 1 (JavaScript foundations), and Lab 2 (Observable basics). You will also need a GitHub account and VS Code.

Check that Node is installed by running this in a terminal:

```bash
node --version
```

You need version 18 or newer. If you get an error instead of a version number, go back to the Lab 0 setup.

The commands in this lab work in the Mac Terminal and in Windows PowerShell. Where Windows needs something different, the lab shows both versions.

## Why data quality matters

A chart is only as good as the data behind it.

Here is an example from the dataset this lab uses. The county records a category for every food facility, but the same category is written in more than one way. Some rows say `Restaurant with Liquor` and others say `Restaurant w/Liquor`. If you group by category without fixing this, one group of restaurants gets split into two smaller bars. The chart looks fine, but the insight is wrong.

Once the names are standardized, the two bars become one and the chart tells the truth. This lab is about building the habits that get you from the first version to the second.

## Part 1: Finding a dataset

### Step 1. Know what makes a good dataset

Before you use any dataset, ask these five questions.

1. **Where does it come from?** Data from government agencies, research institutions, and established nonprofits is usually more trustworthy than data scraped from random websites.
2. **When was it last updated?** Housing prices from 2015 will mislead you today. Check the last updated date and make sure it fits your question.
3. **How was it collected?** Surveys have sampling bias. Sensors have hardware errors. Administrative records have reporting inconsistencies. None of this makes data unusable, but you need to know what you are working with.
4. **Are there missing values?** Every real dataset has gaps. What matters is whether the gaps are random (usually fine) or systematic (a problem, like a sensor that only records during business hours).
5. **Is the documentation clear?** A good dataset comes with a data dictionary, which explains what each column means and what units it uses. You should never have to guess.

### Step 2. Know where to look

Government and civic data

* https://data.gov (U.S. federal open data)
* https://data.census.gov (U.S. Census Bureau)
* https://data.wprdc.org (Western Pennsylvania Regional Data Center, great for local projects)
* https://opendata.cityofnewyork.us (NYC open data)
* https://data.worldbank.org (World Bank development data)

Environment and climate

* https://www.noaa.gov/data (NOAA weather and climate data)
* https://earthdata.nasa.gov (NASA Earth observation data)
* https://openweathermap.org/api (weather API with a free tier)

Health and demographics

* https://wonder.cdc.gov (CDC public health data)
* https://www.who.int/data (World Health Organization)

General

* https://www.kaggle.com/datasets (community shared datasets, so check the original source carefully)
* https://github.com/awesomedata/awesome-public-datasets (a curated list)
* https://datasetsearch.research.google.com (Google's dataset search)

### Step 3. Choose your dataset

Your dataset must have:

* At least 500 rows
* At least 5 columns
* At least one numeric column and at least one category or date column
* Clear documentation, or column names you can understand
* A CSV or JSON download, or a public API

You can use any dataset that meets these requirements. If you are not sure what to pick, we recommend the **Allegheny County Restaurant and Food Facility Inspections** dataset. Every example in this lab uses it, so you can follow the code exactly as written. It lists food inspections across Allegheny County, including inspection dates, facility categories, locations, and whether a reinspection was needed. It has real quality problems to practice on, and it is local to Pittsburgh.

The dataset page, which also has the data dictionary, is here:
https://data.wprdc.org/dataset/allegheny-county-restaurant-food-facility-inspection-violations

The page lists several files with similar names. Download the first one, **Food Facility/Restaurant Inspections**, as a CSV. Its description says it "contains data about all inspections, not just violations." The other files are either violations only or older copies, and their columns will not match this lab.

When we tested this lab in Summer 2026, the file had 9,501 rows covering August 2025 through July 2026. The county adds new inspections over time, so your numbers may be a little different.

These are the columns in the file:

| Column | What it contains |
|---|---|
| `_id` | Row number from the data portal |
| `inspection_id` | ID for one inspection |
| `placard_desc` | Result of the inspection, like `Inspected & Permitted` |
| `facility_name` | Name of the restaurant or food facility |
| `bus_st_date` | Date the business started |
| `facility_type` | Type of facility, like `Food Facility` or `Mobile Food` |
| `category` | Category code and description, like `201-Restaurant with Liquor` |
| `nonprofit` | Whether the facility is a nonprofit (`Yes` or `No`) |
| `num` | Street number |
| `street` | Street name |
| `city` | Mailing city |
| `state` | State |
| `zip_code` | ZIP code |
| `municipal` | Municipality the facility is actually in |
| `inspect_dt` | Date of the inspection |
| `inspection_purpose` | Why the inspection happened, like `Comprehensive` or `Complaint Investigation` |
| `reinspection_need` | Whether this inspection found problems that require a return visit (`Yes` or `No`) |
| `permit_status` | Permit status, like `Active` or `Expired` |

**Using your own dataset?** The steps are the same, but the code will not run exactly as written. Use your own file name in Steps 7 and 9, give your page its own title, and change the column names in each code block to match your data. The checks in Step 10 are patterns you can reuse on any column: missing values, repeated IDs, wrong types, inconsistent spellings, and odd dates. The specific fixes in Step 11, like the restaurant category pairs and the Pittsburgh municipality names, only apply to the inspections data. Your data will have its own problems, and finding them is part of the assignment. The charts in Steps 15 and 16 show the general shape of a bar chart and a line chart, so swap in your own columns there too.

Write down your answers to the five questions from Step 1. They go in your data quality summary in Step 12.

## Part 2: Setting up your project

### Step 4. Create the project

Open a terminal (Terminal on a Mac, PowerShell on Windows) and go to the folder where you keep your class work, for example your Desktop. Then run:

```bash
npx @observablehq/framework@latest create
```

If npm asks `Ok to proceed? (y)`, type `y` and press Enter. You may also see yellow lines that say `npm warn deprecated`. Those are warnings about packages Framework uses behind the scenes, and you can ignore them.

The command asks five questions:

1. **Where should we create your project?** We suggest typing `./67336_Lab5` and pressing Enter, so your folder name matches your GitHub repository. The default, `./hello-framework`, works too. If you keep it, use `hello-framework` whenever this lab tells you to open or `cd` into your project folder.
2. **What should we title your app?** Type `Lab 5`, or any title you like. It appears at the top of your published site.
3. **Include sample files to help you get started?** Press Enter to accept yes. The sample pages are useful to look at while you work, and you will delete them before you publish.
4. **Install dependencies?** Press Enter to accept "Yes, via npm."
5. **Initialize git repository?** Press Enter to accept yes.

When it finishes, you will see a box called "Next steps" with two commands. Run them:

```bash
cd ./67336_Lab5
npm run dev
```

The terminal prints a local address, usually `http://127.0.0.1:3000`. Open it in your browser. You should see a page called Hello Framework with two sample charts, and links to two example pages in the sidebar.

Keep this terminal running for the whole lab. It runs the dev server, which updates your page every time you save. The terminal will keep printing lines that start with `GET`. That is a log of the files your browser is loading, not an error.

### Step 5. Open a second terminal

You need a second terminal for Git commands, and a new terminal window usually does not open inside your project folder. Move into the project first. The easiest way is to type `cd ` (with a space after it), drag the folder where you created the project (example: `67336_Lab5`) from your file browser into the terminal window, and press Enter.

If you type the path yourself and any folder name has a space in it, put the whole path in quotes. 

The following is an example, you need to replace the path with wherever your lab is saved.
On a Mac:

```bash
cd "/Users/yourname/Desktop/Class Work/67336_Lab5"
```

On Windows:

```powershell
cd "C:\Users\yourname\Desktop\Class Work\67336_Lab5"
```

Check that you are in the right place:

```bash
pwd
ls
```

You should see a bunch of files, but specifically make sure you see `package.json` and `src` in the list. Run every Git command in this lab from this terminal.

### Step 6. Put your project on GitHub

1. Go to github.com and sign in.
2. Click the **+** in the top right corner and choose **New repository**.
3. Name it exactly `67336_Lab5`. (or whatever you named your project)
4. Set it to **Private**.
5. Leave everything else unchecked and click **Create repository**.

Next, add your instructors as collaborators. In your new repository, go to **Settings**, then **Collaborators**, then **Add people**. Add these accounts one at a time: `shihongh`, `ygonz174`, and `lillian-zhao`. GitHub may ask for your password before it shows this page.

Now connect your project to the repository. In your second terminal, run these commands one at a time. Replace `YOUR_USERNAME` with your GitHub username.

```bash
git remote add origin https://github.com/YOUR_USERNAME/67336_Lab5.git
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

You do not need `git init`. The create command in Step 4 already set up Git for you.

On Windows, the first push usually opens a window asking you to sign in to GitHub. Sign in there, and the push continues on its own.

**If Git asks for a username and password in the terminal.** GitHub no longer accepts your account password here. Instead it wants a personal access token, which works like a password just for Git.

1. Go to https://github.com/settings/tokens/new. (To click your way there: your profile picture, then Settings, then Developer settings, then Personal access tokens, then Tokens (classic).)
2. Give it a note like `67336`, pick an expiration that lasts through the end of the semester, and check the **repo** box.
3. Click **Generate token** and copy the token right away. GitHub only shows it once.
4. Back in the terminal, type your GitHub username and press Enter. When it asks for a password, paste the token and press Enter. Nothing appears on screen while you paste. That is normal.

Your computer usually remembers the token, so you should only have to do this once.

**If the push fails,** fix the problem and run only `git push -u origin main` again. If you rerun all five commands, Git will say `remote origin already exists` and `nothing to commit`. Both messages are harmless.

## Part 3: Loading and checking your data

### Step 7. Add your dataset

Find the CSV you downloaded in Step 3. It will probably have a long name made of letters and numbers. Rename it to `inspections.csv` and move it into the `src/data` folder inside your project. That folder already exists and has a few sample files in it. You can leave those alone.

To check that the file is in the right place, run this in your second terminal. On a Mac:

```bash
ls -lh src/data/inspections.csv
```

On Windows, leave off `-lh`, because PowerShell does not understand it:

```powershell
ls src/data/inspections.csv
```

You should see the file listed at about 2 MB. On Windows the size shows in bytes under Length, so look for a number around 1,900,000. GitHub rejects files over 100 MB, so if you chose a much larger dataset, ask a TA how to handle it.

### Step 8. Learn how code works on a Framework page

Read this before you write any code. Most problems in this lab come from one of these five rules.

Your page lives in `src/index.md`. Plain text in that file shows up on the page as writing. Code goes in blocks that start with ` ```js ` and end with ` ``` `. Each block runs in your browser, and the page updates every time you save. Results show up on the page, not in the terminal.

The examples below only explain the rules. You will add the real blocks starting in Step 9.

**Rule 1. A block that is a single value shows that value.**

````md
```js
raw.length
```
````

**Rule 2. A block that creates variables shows nothing unless you ask it to.** If a block uses `const`, `function`, or `import`, Framework runs it but shows nothing, even when the last line is a value. Use `display()` to show something.

````md
```js
const rowCount = raw.length;
display(rowCount);
```
````

**Rule 3. Label what you show.** A bare number like `3` on the page means nothing to a reader. Add a label with backticks and `${}`:

````md
```js
display(`Rows: ${rowCount}`);
```
````

**Rule 4. Each name can only be created once on the page.** If two blocks both say `const byMonth = ...`, both blocks break with an error saying the name "is defined more than once." For the same reason, do not import `Plot`, `d3`, or `Inputs`. Framework already provides them on every page.

**Rule 5. Errors show up in red on the page.** The message quotes the code that broke, so you can match it to the block that caused it. When one block breaks, every block that uses its variables breaks too, so fix the first red error you see.

If you want your code to show on the published page above its result, start the block with ` ```js echo ` instead of ` ```js `. This is optional.

### Step 9. Load your data and take inventory

Open your project folder in VS Code and open `src/index.md`. Delete everything in it and replace it with a title for your page:

````md
# Lab 5: Allegheny County Food Inspections
````

The two sample pages are still in the sidebar if you want to see how they were built.

Below the title, add this block to load your data:

````md
```js
const raw = await FileAttachment("data/inspections.csv").csv({ typed: true });
```
````

This block shows nothing on the page. It only creates `raw`, which every other block uses.

`{ typed: true }` tells Framework to guess the type of each column, so dates become dates and numbers become numbers instead of everything staying as text. It also turns empty cells into `null`, which means "nothing here." The guessing is helpful but not perfect, and in Step 10 you will see where it goes wrong.

Now take a first look. Add each of these as its own block:

````md
```js
display(`Rows: ${raw.length}`);
```

```js
Inputs.table(raw.slice(0, 5))
```

```js
Object.keys(raw[0])
```
````

Save and check your browser. You should see a row count of about 9,500, a table with the first five rows, and the list of column names. Click the triangle next to the list to open it.

Look closely at the table. The dates show up as dates, which means they loaded correctly. Now look at `_id`, `inspection_id`, and `num`. They have commas in them, like `1,070,488`, which means they were read as numbers. They are really labels. You would never add up street numbers.

Below these blocks, write a few sentences in plain text (no code block) answering: How many rows are there? What are the columns? Do the types look right? This writing is part of your submitted page.

### Step 10. Find the data quality problems

Add each check below as its own block, under your inventory. After each one, look at the result and think about what it means before you move on.

**Missing values.** Check the columns you plan to use.

````md
```js
const missingNames = raw.filter(d => d.facility_name == null).length;
const missingDates = raw.filter(d => d.inspect_dt == null).length;
const missingCategories = raw.filter(d => d.category == null).length;
display(`Missing facility names: ${missingNames}`);
display(`Missing inspection dates: ${missingDates}`);
display(`Missing categories: ${missingCategories}`);
```
````

Ask yourself: is a missing category a reason to throw away the whole row, or is the rest of the row still useful?

**Duplicates.** A facility showing up many times is normal, because facilities get inspected more than once. A repeated `inspection_id` is different. It means the same inspection was recorded more than once.

````md
```js
const ids = raw.map(d => d.inspection_id);
const extraCopies = ids.length - new Set(ids).size;
display(`Extra rows with a repeated inspection ID: ${extraCopies}`);
```

```js
const repeatedIds = new Set(ids.filter((id, i) => ids.indexOf(id) !== i));
display(Inputs.table(raw.filter(d => repeatedIds.has(d.inspection_id))));
```
````

The second block shows the repeated rows. Look at them and decide whether they are true copies.

**Wrong types.** Check whether columns loaded as the type you expect.

````md
```js
display(`Inspection dates loaded as real dates: ${raw[0].inspect_dt instanceof Date}`);
display(`Categories that loaded as numbers: ${raw.filter(d => typeof d.category === "number").length}`);
```
````

A category should always be text, like `201-Restaurant with Liquor`. If a row has only the code, like `334`, then `{ typed: true }` reads it as a number. Even one value like that will crash your cleaning code in Step 11 unless you handle it.

**Inconsistent categories.** List every category, sorted so similar ones sit next to each other.

````md
```js
const categories = [...new Set(raw.map(d => d.category))].sort();
display(`Different category values: ${categories.length}`);
display(categories);
```
````

Open the list. Each category starts with a number code followed by a description. Look for the same code with two different descriptions, like `201-Restaurant with Liquor` and `201-Restaurant w/Liquor`. Those are the same kind of business written two ways.

**Inconsistent locations.** The `municipal` column holds the municipality each facility is in. Pittsburgh is split into many versions.

````md
```js
const municipals = [...new Set(raw.map(d => d.municipal))].sort();
display(`Different municipality values: ${municipals.length}`);
display(municipals.filter(m => typeof m === "string" && m.includes("PITTSBURGH")));
```
````

You will see names like `PITTSBURGH-101` and `CITY OF PITTSBURGH -WARD 4`, which are parts of the City of Pittsburgh. A chart grouped by municipality would split the city into dozens of small bars. Look carefully, though. Not every name on this list is part of the city.

Also compare `city` and `municipal` for a few rows in your table. The `city` column is the mailing address, which is not always the municipality the facility is actually in.

**Outliers and placeholder values.** Check the business start dates.

````md
```js
const startDates = raw.map(d => d.bus_st_date).filter(d => d instanceof Date);
const januaryFirsts = startDates.filter(d => d.getUTCMonth() === 0 && d.getUTCDate() === 1).length;
display(`Earliest start date: ${d3.min(startDates).toISOString().slice(0, 10)}`);
display(`Latest start date: ${d3.max(startDates).toISOString().slice(0, 10)}`);
display(`Missing start dates: ${raw.length - startDates.length}`);
display(`Start dates on January 1st: ${januaryFirsts} of ${startDates.length}`);
```
````

A very old start date could be real for a business that has been open a long time. The January 1st count is more interesting. If start dates were spread evenly across the year, only about 1 in 365 would land on January 1st. A much higher count suggests January 1st is a placeholder that means "we only know the year."

Why `.toISOString().slice(0, 10)`? The file stores each date at midnight UTC. Your browser shows dates in Eastern time, which turns midnight into the evening before, so every date would look one day early. `.toISOString().slice(0, 10)` shows the date exactly as it is written in the file.

**Date range.** Find out what time period your data covers.

````md
```js
const [firstInspection, lastInspection] = d3.extent(raw, d => d.inspect_dt);
display(`Inspections from ${firstInspection.toISOString().slice(0, 10)} to ${lastInspection.toISOString().slice(0, 10)}`);
```
````

If the data starts or ends partway through a month, that month will look low in any monthly chart. Keep that in mind for your caveats.

You can check any other column with the same pattern as the categories block. For example, `permit_status` and `inspection_purpose` have their own spelling problems. Give each new list its own name, like `const statuses = ...`, so you do not break Rule 4.

### Step 11. Clean your data

Now fix what you found. Write down each decision and the reason for it, because they go in your summary in Step 12.

The first four blocks in this step only create variables, so nothing new appears on the page until you add the before and after block.

**Remove rows that are missing critical values.** A row with no facility name or no inspection date cannot be counted or placed in time, so it goes.

````md
```js
const cleaned = raw.filter(d => d.facility_name != null && d.inspect_dt != null);
```
````

Since `{ typed: true }` turns empty cells into `null`, checking for `null` also catches blank cells. Notice that this keeps rows with a missing category. Those are still real inspections, so the next steps label them `Unknown` instead of deleting them. You could decide differently, as long as you explain why.

**Remove duplicate inspections.** Keep the first copy of each `inspection_id` and drop the rest.

````md
```js
const seen = new Set();
const deduped = cleaned.filter(d => {
  if (seen.has(d.inspection_id)) return false;
  seen.add(d.inspection_id);
  return true;
});
```
````

**Write your cleaning rules.** This block sets up three helpers.

`categoryFixes` is a lookup table. Each line says "this spelling means the same thing as that one." We filled in the restaurant pairs and the `334` code. Using your sorted category list from Step 10, find the other pairs and add them yourself.

`cleanCategory` cleans one category value. It labels a missing category `Unknown`, turns the value into text with `String()` so a number like `334` cannot crash it, removes the number code at the front, and then checks your lookup table. The line with `split("-").slice(1).join("-")` cuts the text at every dash, drops the first piece (the code), and glues the rest back together. The gluing matters because some descriptions have a dash of their own, like `Social Club-Bar Only`.

`isPittsburgh` decides whether a municipality name belongs to the City of Pittsburgh. It checks how the name starts instead of whether it contains "PITTSBURGH" anywhere, because `EAST PITTSBURGH` is a separate borough.

````md
```js
const categoryFixes = {
  "Restaurant w/Liquor": "Restaurant with Liquor",
  "Restaurant w/o Liquor": "Restaurant without Liquor",
  "Chain Restaurant w/Liquor": "Chain Restaurant with Liquor",
  "Chain Restaurant w/o Liquor": "Chain Restaurant without Liquor",
  "334": "Temporary Fish Fry"
};

function cleanCategory(category) {
  if (category == null) return "Unknown";
  const text = String(category).trim();
  const name = text.includes("-") ? text.split("-").slice(1).join("-").trim() : text;
  return categoryFixes[name] ?? name;
}

function isPittsburgh(name) {
  return typeof name === "string" &&
    (name.startsWith("PITTSBURGH") || name.startsWith("CITY OF PITTSBURGH"));
}
```
````

**Apply the rules.** This creates `normalized`, the cleaned dataset your charts will use. It keeps every original column, trims and lowercases facility names so `Ali Baba` and `ALI BABA ` count as one name, and adds two new columns, `category_clean` and `municipal_clean`.

````md
```js
const normalized = deduped.map(d => ({
  ...d,
  facility_name: String(d.facility_name).trim().toLowerCase(),
  category_clean: cleanCategory(d.category),
  municipal_clean: isPittsburgh(d.municipal) ? "PITTSBURGH" : d.municipal
}));
```
````

**Compare before and after.**

````md
```js
display(`Raw rows: ${raw.length}. Cleaned rows: ${normalized.length}. Rows removed: ${raw.length - normalized.length}.`);
```
````

You should only lose a handful of rows. If you lose hundreds, check your filters.

Once you have added your own category fixes, list the clean categories to make sure every pair was merged:

````md
```js
display([...new Set(normalized.map(d => d.category_clean))].sort());
```
````

### Step 12. Write your data quality summary

Before you build any charts, write a short summary as plain text in `index.md`, below your cleaning blocks. Answer these four questions:

1. What dataset did you choose, and where is it from? Include your answers to the five questions from Step 1.
2. What quality problems did you find?
3. What did you do about each one, and why?
4. How many rows did you lose during cleaning? Is that acceptable?

You can put live numbers in your writing with `${}`. For example, writing `We removed ${raw.length - normalized.length} rows.` in plain text shows the actual number on the page.

A chart without documentation of its data quality is incomplete. 

## Part 4: Choosing the right visualization

### Step 13. Match your question to a chart type

The most common mistake in data visualization is choosing a chart type first and then forcing the data into it. Work the other way around. Start with the question, then find the chart that answers it most clearly.

| If you want to show | Consider | Avoid |
|---|---|---|
| How a value changes over time | Line chart, area chart | Pie chart |
| How values compare across categories | Bar chart, dot plot | Line chart, which suggests a connection between categories that is not there |
| How two numeric variables relate | Scatter plot | Bar chart |
| How values are distributed | Histogram, box plot | Line chart |
| Where things are located | Map (choropleth, dot map, heatmap) | Bar chart |
| Parts of a whole | Stacked bar, treemap, pie (only with a few categories) | Line chart |

Ask yourself: if someone who had never seen this data looked at my chart for ten seconds, would they get the main point? If not, the chart type is probably wrong, or the data needs more cleaning.

### Step 14. Plan your two visualizations

You will build two charts from your cleaned data. They should answer two different questions about the same dataset.

Before writing any code, write this down in `index.md` as plain text, once for each chart:

1. **Question:** What am I trying to show?
2. **Chart type:** What type will I use, and why?
3. **X axis:** What goes here?
4. **Y axis:** What goes here, if anything?
5. **Color or size:** Will I show a third variable?

Here are the two examples this lab builds:

1. "Which kinds of facilities most often need a reinspection?" A horizontal bar chart of the reinspection rate for each category, sorted from highest to lowest.
2. "How did the number of inspections change from month to month?" A line chart with the month on the x axis.

Notice that the first question asks how often, which calls for a rate (the share of inspections that needed a reinspection), not a count. Restaurants have the most reinspections partly because there are so many restaurants. A rate lets you compare a big category with a small one fairly.

## Part 5: Building your visualizations

### Step 15. Build Visualization 1

This chart groups the inspections by clean category. For each category, it counts the inspections and works out what share of them needed a reinspection. It only keeps categories with at least 50 inspections, because a category with 2 inspections and 1 reinspection would show 50 percent, which tells you almost nothing.
````md
```js
const reinspectionRates = d3.rollups(
  normalized,
  v => ({
    inspections: v.length,
    rate: v.filter(d => d.reinspection_need === "Yes").length / v.length
  }),
  d => d.category_clean
)
  .map(([category, stats]) => ({ category, ...stats }))
  .filter(d => d.inspections >= 50);

display(Plot.plot({
  title: "Share of inspections that needed a reinspection",
  marginLeft: 220,
  x: { label: "Inspections needing reinspection (%)", percent: true, grid: true },
  y: { label: null },
  marks: [
    Plot.barX(reinspectionRates, {
      x: "rate",
      y: "category",
      fill: "steelblue",
      sort: { y: "-x" },
      channels: { Inspections: "inspections" },
      tip: true
    }),
    Plot.ruleX([0])
  ]
}));
```
````

Hover over a bar to see its exact rate and how many inspections it is based on. You will see a bar called `Unknown` for the rows with a missing category. Decide whether to keep it and explain your choice in your caveat.

After you build it, ask yourself: does this chart clearly answer the question from Step 14? If not, change the chart type, axes, or filters before moving on.

### Step 16. Build Visualization 2

This chart groups the inspections by month and counts them. `d3.utcMonth` rounds each date down to the first day of its month. It uses the UTC version because the dates are stored in UTC. The regular `d3.timeMonth` uses your local time zone, which pushes every inspection from the 1st of a month into the month before. The months are then sorted, because Plot connects the dots in the order the data is listed, not from left to right. Without the sort, the line zigzags back and forth.
````md
```js
const byMonth = d3.rollups(
  normalized,
  v => v.length,
  d => d3.utcMonth(d.inspect_dt)
)
  .map(([month, count]) => ({ month, count }))
  .sort((a, b) => a.month - b.month);

display(Plot.plot({
  title: "Inspections per month",
  x: { label: "Month" },
  y: { label: "Inspections", grid: true, zero: true },
  marks: [
    Plot.lineY(byMonth, { x: "month", y: "count", stroke: "steelblue", strokeWidth: 2 }),
    Plot.dot(byMonth, { x: "month", y: "count", fill: "steelblue", tip: true })
  ]
}));
```
````

Your second chart should use a different chart type and answer a different question than your first. If your first chart compares categories, your second could show change over time, like this one, or a distribution, like a histogram. For other chart types, see the Observable Plot documentation at https://observablehq.com/plot/.

### Step 17. Add written context

A chart without context is incomplete. Under each chart, add plain text that covers:

1. The question the chart answers
2. The key insight, in one sentence a non expert could understand
3. A caveat, meaning a limitation the reader should know about

Here is an example for the monthly chart, based on the data when we tested it:

> **What this shows:** The county ran the most inspections in September 2025 (980) and the fewest in December 2025 (691), not counting August.
>
> **Caveat:** The data starts on August 11, 2025, so August only includes part of the month and looks lower than it really was. The data also covers only one year, so it cannot show long term trends.

## Part 6: Reflection

### Step 18. Write your reflection

At the end of your page, add a section called "Reflection" and answer all four questions in 2 to 3 sentences each.

1. **What was the messiest part of your data?** Describe the biggest quality problem you found and how you handled it.
2. **Did cleaning change your conclusions?** Compare what the raw data seemed to show with what the cleaned data shows. One way to test this: build your first chart again using `d.category` instead of `d.category_clean` and see what changes.
3. **What would you do with more time?** What other cleaning, filtering, or added data would improve your analysis?
4. **Why did you choose these two chart types?** Explain why each chart type fits its question better than the alternatives.

## Part 7: Publishing

### Step 19. Clean up before you publish

1. Delete `src/example-dashboard.md` and `src/example-report.md` so the sample pages do not appear on your site.
2. Open `observablehq.config.js` and check that the `title` line has your title, not "Hello Framework."
3. Scroll through your whole page in the browser and make sure there are no red errors.

### Step 20. Build the site

In your second terminal, run:

```bash
npm run build
```

This creates a `dist` folder with your finished site. When it works, it ends with a short table listing your pages. If you see an error, fix it before you move on.

A clean build does not prove your charts work. The build puts your page together, but your code blocks only run in the browser. Always check the page itself too.

### Step 21. Commit and push

```bash
git add .
git commit -m "Finish Lab 5"
git push
```

### Step 22. Deploy to Vercel

1. Go to https://vercel.com and sign in with your GitHub account.
2. Click **Add New**, then **Project**.
3. Find `67336_Lab5` in the list and click **Import**. If it is not listed, Vercel does not have access to your private repository yet. Use the link under the list to adjust Vercel's GitHub permissions, give it access to `67336_Lab5`, and come back.
4. For **Framework Preset**, choose **Other**.
5. Leave **Root Directory** as it is.
6. Open **Build and Output Settings**. Set **Build Command** to `npm run build` and **Output Directory** to `dist`. If a field is grayed out, turn on the **Override** switch next to it first.
7. Click **Deploy**.

When it finishes, go to your project's page on Vercel. Under **Domains** you will find a short link that ends in `.vercel.app`. That is the link to submit. Vercel also shows longer links with extra letters and numbers, and those can require a Vercel login to open.

Open your link in a private or incognito window. It should load without asking you to log in, and both charts should appear.

From now on, every time you push to GitHub, Vercel rebuilds your site automatically.

## Submission

Submit these on Canvas:

1. Your GitHub repository link, like `https://github.com/YOUR_USERNAME/67336_Lab5`
2. Your Vercel link, ending in `.vercel.app`

Make sure `shihongh`, `ygonz174`, and `lillian-zhao` are collaborators on your repository before you submit.

## Troubleshooting

| What you see | What it means | What to do |
|---|---|---|
| `running scripts is disabled on this system` when you run `npm` or `npx` on Windows | PowerShell is blocking the script that starts npm | Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, answer `Y`, then close and reopen PowerShell |
| `A parameter cannot be found that matches parameter name` on Windows | PowerShell does not use the same options as Mac commands | Use the Windows version of the command shown in that step |
| `LF will be replaced by CRLF` when you use Git on Windows | Windows and Mac end lines of text differently, and Git is converting them | Nothing is wrong. You can ignore it |
| A block shows nothing | The block creates variables (Rule 2) | Wrap what you want to show in `display()` |
| `Cannot read properties of null` | Your code treated a blank value as text | Check for `null` first, like `cleanCategory` does |
| `... is not a function` | A value is not the type you expected, often a number where you expected text | Wrap it in `String()` first |
| `... is defined more than once` | Two blocks create the same name | Rename one, or delete the extra `import` |
| The line chart zigzags | The data is not sorted by date | Add `.sort((a, b) => a.month - b.month)` |
| Dates look one day early | Your browser converts UTC dates to Eastern time | Use `.toISOString().slice(0, 10)` to show dates and `d3.utcMonth` to group them |
| `git push` asks for a password | GitHub wants a personal access token | Follow the token steps in Step 6 |
| `remote origin already exists` or `nothing to commit` | You ran a command a second time | Nothing is wrong. Run only the push command |
| `Please tell me who you are` when you commit | Git does not know your name yet | Run `git config --global user.name "Your Name"` and `git config --global user.email "you@example.com"`, then commit again |
| Your Vercel link asks you to log in | You opened one of the longer deployment links | Use the short link under Domains |
| The site works on your computer but not on Vercel | Something is missing from the deploy | Check the build settings from Step 22, and check that `src/data/inspections.csv` is in your GitHub repository |

## Quick reference: cleaning patterns

| Problem | How to find it | How to fix it |
|---|---|---|
| Missing values | `raw.filter(d => d.col == null).length` | Remove the row, or label it (like `Unknown`) and explain why |
| Duplicates | Compare `ids.length` with `new Set(ids).size` | Keep the first copy with a `seen` set |
| Wrong type | `typeof d.col` or `d.col instanceof Date` | `String()`, `Number()`, or `new Date()` |
| Inconsistent text | List unique values with `[...new Set(...)].sort()` | `.trim()`, `.toLowerCase()`, or a lookup table |
| Placeholder values | Check the min, the max, and values that repeat suspiciously often | Filter to a plausible range, or flag them and explain |
| Dates one day off | Compare with the raw file | `.toISOString()` to show dates, `d3.utcMonth` to group them |

## Quick reference: chart guide

| Question | Best chart |
|---|---|
| How does something change over time? | Line chart |
| How do categories compare? | Bar chart, sorted |
| How are two numbers related? | Scatter plot |
| Where is this happening? | Choropleth map or dot map |
| How is a value distributed? | Histogram or box plot |
| What share of the whole is each part? | Stacked bar or treemap |
