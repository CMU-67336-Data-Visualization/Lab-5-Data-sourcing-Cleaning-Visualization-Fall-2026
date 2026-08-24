# Lab 5: Data Sourcing, Cleaning & Visualization

**67-336 Data Visualization | Fall 2026**

---

## Overview

This lab ties directly into the AI & Data Cleaning lecture. You will go through the full pipeline that real data analysts use every day: finding a dataset, evaluating its quality, cleaning it, and choosing the right visualization for what the data actually says.

By the end you will have a cleaned dataset and at least two visualizations deployed on Vercel. You will understand why you made the choices you did, not just how.

---

## Before You Start

If you need a refresher on Observable Framework, go back and review the Lab 2 instructions before continuing. If your Lab 2 setup is no longer working, re-run `npm install` in that project folder before continuing here.

> Quick refresher on how Observable Framework works: every code block (fenced with ```` ```js ````) inside a `.md` page runs live in your browser and re-renders automatically whenever you save the file. There is no separate "run this script" command — you write code in `src/index.md`, save, and watch the result update at `localhost:3000` in real time. Keep that dev server terminal running the whole time you work.

---

## Learning Objectives

By the end of this lab, you will be able to:

- Find and evaluate real world datasets from reliable sources
- Identify common data quality problems and fix them
- Make intentional decisions about which chart type fits your data
- Build visualizations in Observable from cleaned data
- Explain the difference between a visualization made from dirty data vs. clean data
- Deploy your finished notebook to Vercel

---

## Why Data Quality Matters

Before writing a single line of code, here is the key idea for this lab.

**A visualization is only as good as the data behind it.**

Consider these two scenarios:

**Scenario A:** A dataset of restaurant inspections where the same restaurant appears under slightly different names ("Ali Baba," "ALI BABA," "Ali Baba Restaurant") and the same municipality is written three different ways. A chart grouping by restaurant name or location will split one business into three, making small restaurants look bigger and big ones look smaller. The chart looks fine, but the insight is wrong.

**Scenario B:** The same dataset, cleaned. Names standardized, duplicates removed, inconsistent categories fixed. Now the chart tells the truth.

This lab is about building the habits that get you from Scenario A to Scenario B!

---

## Part 1: Finding a Dataset

---

### Step 1: What makes a good dataset?

Not all data is created equal. Before you use a dataset, ask yourself these five questions:

**1. Where does it come from?**
Data from government agencies, research institutions, and established nonprofits is generally more trustworthy than data scraped from random websites or crowdsourced without oversight.

**2. When was it last updated?**
A dataset about housing prices from 2015 will mislead you today. Check the last updated date and make sure it fits your question.

**3. How was it collected?**
Survey data has sampling bias. Sensor data has hardware error. Administrative data has reporting inconsistencies. None of this makes data unusable, but you need to know what you are working with.

**4. Are there missing values?**
Every real dataset has gaps. The question is whether the gaps are random (usually okay) or systematic (a problem, e.g. a sensor that only records during business hours).

**5. Is the documentation clear?**
A dataset without a data dictionary (a guide explaining what each column means and what units are used) is a red flag. You should never have to guess what a column means.

---

### Step 2: Where to find datasets

Here are reliable sources to use for this lab and your future projects:

**Government and civic data:**
- https://data.gov — U.S. federal open data portal
- https://data.census.gov — U.S. Census Bureau
- https://data.wprdc.org — Western Pennsylvania Regional Data Center (great for local projects)
- https://opendata.cityofnewyork.us — NYC open data
- https://data.worldbank.org — World Bank global development data

**Environmental and climate:**
- https://www.noaa.gov/data — NOAA weather and climate data
- https://earthdata.nasa.gov — NASA Earth observation data
- https://openweathermap.org/api — Weather API (free tier available)

**Health and demographics:**
- https://wonder.cdc.gov — CDC public health data
- https://www.who.int/data — World Health Organization

**General purpose:**
- https://www.kaggle.com/datasets — Community shared datasets (check the source carefully)
- https://github.com/awesomedata/awesome-public-datasets — Curated list of public datasets
- https://datasetsearch.research.google.com — Google's dataset search engine

---

### Step 3: Choose your dataset

For this lab, choose one dataset from the sources above. It must meet all of the following criteria:

- [ ] At least 500 rows
- [ ] At least 5 columns
- [ ] Contains at least one numeric column and one categorical or date column
- [ ] Has clear documentation or column headers you can interpret
- [ ] Is available as a CSV, JSON, or via a public API

> Not sure what to pick? We recommend the **Allegheny County Restaurant and Food Facility Inspection** dataset from https://data.wprdc.org/dataset/allegheny-county-restaurant-food-facility-inspection-violations. It contains inspection records for restaurants and food facilities across Allegheny County including inspection dates, facility types, categories, locations, and permit status. It has real data quality issues to practice cleaning on and is locally relevant to Pittsburgh and CMU students. All examples in this lab use this dataset.
>
> Heads up: that WPRDC page lists several similarly-named files under "Data and Resources" (e.g. "Food Facility/Restaurant Inspections," "Food Facility/Restaurant Inspection Violations...," "Food Facility/Restaurant Inspections (2014–2025)," a "(deprecated)" version, etc). **Download the first one — "Food Facility/Restaurant Inspections"** ("Contains data about all inspections, not just violations"). The other entries are either a subset (violations only) or an older/deprecated copy, and won't match the column reference table in Step 6.

Write down your answers to the five quality questions from Step 1 before moving on. You will reference them in your write-up (see the note in Step 10 about exactly where to put this).

---

## Part 2: Setting Up Your Project

---

### Step 4: Create a new Observable Framework project

Create a new folder for this lab and initialize an Observable Framework project:

```bash
mkdir lab5-data-pipeline
cd lab5-data-pipeline
npm init @observablehq/framework@latest .
```

When prompted, accept the defaults.

> If that command fails or hangs for you, use this alternative instead:
> ```bash
> npm install -g @observablehq/framework
> observable create
> ```
> Follow the prompts, then make sure you `cd` into whatever folder name it creates before continuing.

Then start the dev server:

```bash
npm run dev
```

> Make sure you're inside the project folder (`lab5-data-pipeline`, or whatever `observable create` named it) before running this — if you get a "no such file" or "package.json not found" error, run `cd lab5-data-pipeline` (or `ls` to check the folder name) first.

Open the URL shown in your terminal (usually `http://localhost:3000`) in **Chrome**. You should see the default Observable Framework page — it will already have example content (a sample chart or two) already on it. That's expected; you'll replace it in Step 7.

> Tip: Keep this terminal window running the dev server. Open a second terminal window for any other commands like git.

---

### Step 5: Set up your GitHub repo

1. Go to github.com and sign in.
2. Click the **+** icon in the top right corner and choose "New repository".
3. Name it exactly:
```
67336_Lab5
```
4. Set visibility to **Private**.
5. Leave everything else unchecked.
6. Click "Create repository".

Then add your instructors as collaborators:

7. Go to your new repo → **Settings** → **Collaborators** → **Add people**
8. Add each of the following one at a time:
   - `shihongh`
   - `ygonz174`
   - `lillian-zhao`

Connect your local project to GitHub:

```bash
git init
git remote add origin https://github.com/YOUR-USERNAME/67336_Lab5.git
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## Part 3: Loading and Evaluating Your Data

---

### Step 6: Add your dataset

Download your dataset as a CSV file and place it inside the `src/data/` folder of your project.

For the Allegheny County Restaurant Inspections dataset:

1. Go to https://data.wprdc.org/dataset/allegheny-county-restaurant-food-facility-inspection-violations
2. Under "Data and Resources," click the **Download** button on the **first entry ("Food Facility/Restaurant Inspections")** and select **CSV**. (See the note in Step 3 — don't grab one of the other similarly-named files.)
3. Save the file as `inspections.csv` and place it in `src/data/` (create the `data` folder inside `src` if it doesn't already exist).

Your dataset has the following columns:

| Column | What it contains |
|---|---|
| `_id` | Unique row identifier |
| `inspection_id` | Inspection identifier |
| `placard_desc` | Inspection result (e.g. "Inspected & Permitted") |
| `facility_name` | Name of the restaurant or food facility |
| `bus_st_date` | Business start date |
| `facility_type` | Type of facility |
| `category` | Category code and description (e.g. "201-Restaurant with Liquor") |
| `nonprofit` | Whether the facility is a nonprofit (Yes or No) |
| `num` | Street number |
| `street` | Street name |
| `city` | City |
| `state` | State |
| `zip_code` | Zip code |
| `municipal` | Municipality name |
| `inspect_dt` | Date of inspection |
| `inspection_purpose` | Comprehensive, Reinspection, or Service Request |
| `reinspection_need` | Whether a reinspection is needed (Yes or No) |
| `permit_status` | Active, About to Expire, etc. |

---

### Step 7: Load your data and take inventory

Open `src/index.md` in VS Code. **This is your main notebook file, and Observable Framework pre-fills it with placeholder/example content** (a demo chart or two, sample text) when the project is created — you didn't do anything wrong if it already has stuff in it. You have two options:

- **Delete the placeholder content** and start fresh, or
- **Scroll to the bottom and add your own new section below it** (recommended, so you always have the working example to reference).

Either way, this is the one file you'll be editing for the rest of the lab.

Add a new fenced JavaScript code block to `src/index.md` to load your dataset:

````
```js
const raw = await FileAttachment("data/inspections.csv").csv({ typed: true });
```
````

> Note: The `{ typed: true }` flag tells Observable to automatically detect column types — numbers as numbers, dates as dates — rather than reading everything as strings. Always include this.

**How this actually runs:** there's no separate command to execute this — as soon as you save `src/index.md`, the dev server (from Step 4) picks up the change and re-renders the page automatically at `localhost:3000`. Any value you reference in a code block (like `raw.length` below) gets displayed inline on the page itself, in the browser — not printed to your terminal. Keep the browser tab open next to VS Code so you can see results the moment you save.

Take a first look at your data by adding these cells (each ```` ```js ```` fence is its own cell):

```js
// How many rows?
raw.length
```

```js
// What do the first few rows look like?
raw.slice(0, 5)
```

```js
// What columns do you have?
Object.keys(raw[0])
```

Save the file and check the browser — you should see the row count, a preview table, and a list of column names appear on the page.

**Write down what you see** by adding a plain Markdown cell (no ```` ```js ```` fence, just regular text) directly below these code blocks in `index.md`, answering: How many rows? What are the column names? Do the types look right? This is part of your submitted notebook — you're not writing it anywhere else, it lives on the page alongside your code and charts.

> Note: If you chose a different dataset, replace the column names in the examples below with your actual column names.

---

### Step 8: Identify data quality problems

Real data almost always has at least some of these issues. Work through each one for your dataset, adding each as its own code block in `index.md` below the inventory cells from Step 7.

**Missing values:**
```js
// Check for missing facility names
const missingNames = raw.filter(d => d.facility_name == null || d.facility_name === "").length;
missingNames
```
```js
// Check for missing inspection dates
const missingDates = raw.filter(d => d.inspect_dt == null || d.inspect_dt === "").length;
missingDates
```

> Tip: in Observable, just put the variable name as the last line of a cell (like `missingNames` above) to display it — you don't need `console.log` here since the value renders directly on the page.

**Duplicates:**
```js
// The same facility can appear multiple times — once per inspection.
// Check for duplicate inspection IDs, which would be a true duplicate.
const ids = raw.map(d => d.inspection_id);
const uniqueIds = new Set(ids);
const duplicates = ids.length - uniqueIds.size;
duplicates
```

**Inconsistent formatting:**
```js
// Check how municipality names are written
const municipals = [...new Set(raw.map(d => d.municipal))];
municipals
```

You will likely see the same municipality written multiple ways, for example `"PITTSBURGH-101"`, `"PITTSBURGH-102"`, and `"CITY OF PITTSBURGH -WARD 4"` all referring to Pittsburgh. That breaks grouping and counts.

```js
// Check how categories are written
const categories = [...new Set(raw.map(d => d.category))];
categories
```

Notice that categories combine a numeric code and a description, for example `"201-Restaurant with Liquor"`. You may want to split these or standardize them.

**Wrong data types:**
```js
// Check if inspection date came in as a string or a Date
typeof raw[0].inspect_dt
```

**Outliers:**
```js
// Check business start dates — some go back to the early 1900s
import { min, max } from "npm:d3-array";
const dates = raw.map(d => new Date(d.bus_st_date)).filter(d => !isNaN(d));
`Earliest: ${min(dates)}, Latest: ${max(dates)}`
```

A business start date of 1931 is unusual but plausible for an old establishment. A date of 1900-01-01 for many records is likely a sentinel value used when the real date was unknown.

---

### Step 9: Clean your data

Now fix the problems you found. Document every decision you make — this is part of your write-up.

**Remove rows with missing values in critical columns:**
```js
const cleaned = raw.filter(d =>
  d.facility_name != null &&
  d.facility_name !== "" &&
  d.inspect_dt != null &&
  d.inspect_dt !== ""
);
```

**Remove duplicate inspections:**
```js
const seen = new Set();
const deduped = cleaned.filter(d => {
  if (seen.has(d.inspection_id)) return false;
  seen.add(d.inspection_id);
  return true;
});
```

**Standardize facility names, clean categories, and convert dates (do this in one pass so later steps have everything they need):**
```js
const normalized = deduped.map(d => ({
  ...d,
  facility_name: d.facility_name.trim().toLowerCase(),
  category_clean: d.category.includes("-")
    ? d.category.split("-").slice(1).join("-").trim()
    : d.category.trim(),
  inspect_date: new Date(d.inspect_dt)
}));
```

**After cleaning, compare row counts:**
```js
`Raw rows: ${raw.length}, Cleaned rows: ${normalized.length}, Rows removed: ${raw.length - normalized.length}`
```

---

### Step 10: Write your data quality summary

Before building any charts, add a Markdown cell (plain text, no code fence) to your notebook answering these four questions. This is the same "write it down" location referenced in Step 7 — everything goes directly into `index.md`, there is no separate document to submit for this part.

1. What dataset did you choose and where is it from?
2. What quality issues did you find?
3. What did you decide to do about each one, and why?
4. How many rows did you lose during cleaning? Is that acceptable?

This matters because a visualization without documentation of its data quality is incomplete. In professional settings, this summary is often the first thing a reviewer reads.

---

## Part 4: Choosing the Right Visualization

---

### Step 11: Match your question to a chart type

The most common mistake in data visualization is choosing a chart type first and then forcing the data into it. Do it the other way around: start with the question, then find the chart that answers it most clearly.

| If you want to show... | Consider... | Avoid... |
|---|---|---|
| How a value changes over time | Line chart, area chart | Pie chart, bar chart |
| How values compare across categories | Bar chart, dot plot | Line chart |
| How two numeric variables relate | Scatter plot | Bar chart |
| How data is distributed | Histogram, box plot | Line chart |
| Where things are located | Map (choropleth, dot map, heatmap) | Bar chart |
| Part to whole relationships | Stacked bar, treemap, pie (only with few categories) | Line chart |

**The question to ask yourself:** If a person who had never seen this data looked at my chart for ten seconds, would they understand the key insight?

If the answer is no, the chart type is probably wrong, or the data needs more cleaning.

---

### Step 12: Plan your two visualizations

You will build two visualizations from your cleaned dataset. They should answer two different questions about the same data.

Before writing any code, write down the following for each visualization (as another Markdown cell in `index.md`):

**Visualization 1:**
- Question: What am I trying to show?
- Chart type: What type will I use and why?
- X axis: What goes here?
- Y axis: What goes here (if applicable)?
- Color/size: Will I encode a third variable?

**Visualization 2:**
- Same fields as above, for a different question

**Example using the Restaurant Inspections data:**
- Viz 1: "Which facility categories get reinspected most often?" → Bar chart sorted by reinspection count
- Viz 2: "How has the number of inspections changed over time?" → Line chart with inspection date on the x axis

---

## Part 5: Building Your Visualizations

---

### Step 13: Build Visualization 1

Use Observable Plot to build your first chart, in a new code block in `index.md`. Here are the patterns you will use most:

**Bar chart:**
```js
import * as Plot from "npm:@observablehq/plot";
import * as d3 from "npm:d3";

// Count reinspections by category
const reinspections = d3.rollups(
  normalized.filter(d => d.reinspection_need === "Yes"),
  v => v.length,
  d => d.category_clean
).map(([category, count]) => ({ category, count }));

Plot.plot({
  marks: [
    Plot.barX(reinspections, {
      y: "category",
      x: "count",
      fill: "steelblue",
      sort: { y: "-x" }
    })
  ],
  y: { label: "Facility Category" },
  x: { label: "Number of Reinspections" },
  title: "Reinspections Needed by Facility Category",
  marginLeft: 200
})
```

**Line chart (time series):**
```js
// Count inspections per month
const byMonth = d3.rollups(
  normalized,
  v => v.length,
  d => d3.timeMonth(d.inspect_date)
).map(([date, count]) => ({ date, count }));

Plot.plot({
  marks: [
    Plot.lineY(byMonth, {
      x: "date",
      y: "count",
      stroke: "steelblue",
      strokeWidth: 2
    }),
    Plot.dot(byMonth, {
      x: "date",
      y: "count",
      fill: "steelblue",
      r: 3
    })
  ],
  x: { label: "Month", type: "time" },
  y: { label: "Number of Inspections" },
  title: "Inspections Over Time"
})
```

**Bar chart comparing permit status:**
```js
const byStatus = d3.rollups(
  normalized,
  v => v.length,
  d => d.permit_status
).map(([status, count]) => ({ status, count }));

Plot.plot({
  marks: [
    Plot.barY(byStatus, {
      x: "status",
      y: "count",
      fill: "steelblue",
      sort: { x: "-y" }
    })
  ],
  x: { label: "Permit Status" },
  y: { label: "Number of Facilities" },
  title: "Facilities by Permit Status"
})
```

After building, ask yourself: does this chart clearly answer the question I defined in Step 12? If not, adjust the chart type, axes, or filters before moving on.

---

### Step 14: Build Visualization 2

Build your second chart using the same process, in a new code block below Visualization 1. It should ask a different question than Visualization 1.

If your first chart was a comparison (bar chart), consider making your second one show change over time (line chart) or distribution (histogram). Variety helps demonstrate that you understand when to use different chart types.

---

### Step 15: Add written context to your notebook

A chart without context is incomplete. Underneath each visualization (as a Markdown cell, same as before), add:

1. **The question** this chart answers
2. **The key insight** — what does the chart actually show? Write one sentence that a non-expert could understand.
3. **A caveat** — what limitation should the reader know?

**Example:**

> **What this shows:** Social clubs and bars have the highest reinspection rates of any facility category, suggesting they receive more violations on initial inspection than restaurants.
>
> **Caveat:** This reflects the number of inspections recorded, not the severity of violations. A facility with many minor violations may appear more often than one with a single critical violation.

---

## Part 6: Reflection

---

### Step 16: Write your reflection

In your notebook, add a final Markdown section called "Reflection" and answer all four questions. Aim for 2 to 3 sentences each.

**1. What was the messiest part of your data?**
Describe the most significant quality issue you found and how you handled it.

**2. Did cleaning change your conclusions?**
Compare what the raw data seemed to show vs. what the cleaned data showed. Did any patterns disappear or appear after cleaning?

**3. What would you do differently with more time?**
What additional cleaning, filtering, or enrichment would improve your analysis?

**4. Why did you choose these two chart types?**
Explain specifically why each chart type fits its question better than the alternatives.

---

## Part 7: Deployment

---

### Step 17: Commit your work

Before deploying, make sure everything is committed:

```bash
git add .
git commit -m "Complete Lab 5 — data pipeline and visualizations"
git push origin main
```

---

### Step 18: Build your project

```bash
npm run build
```

This generates a `dist/` folder with your compiled notebook.

---

### Step 19: Deploy to Vercel

1. Go to https://vercel.com and sign in with your GitHub account.
2. Click **"Add New Project"**.
3. Select your `67336_Lab5` repository.
4. For **Framework Preset**, choose **Other**.
5. Set the following:
   - **Root Directory:** `./` (or leave blank)
   - **Build Command:** `npm run build`
   - **Output Directory:** `dist`
6. Click **Deploy**.

After deployment finishes, you will get a live URL like:
```
https://67336-lab5-yourname.vercel.app
```

Open it and confirm both visualizations render correctly in the browser.

> Tip: If charts look different in the browser than in your dev server, check for any hardcoded file paths. Observable Framework handles file attachments differently in production. Always use `FileAttachment()` rather than raw `fetch()` calls for local files.

---

## Submission

Submit the following on Canvas:

- Your **GitHub repo link** (e.g. `https://github.com/YOUR-USERNAME/67336_Lab5`)
- Your **Vercel live site link** (e.g. `https://67336-lab5-yourname.vercel.app`)

> WARNING: Make sure you have added `shihongh`, `ygonz174`, and `lillian-zhao` as collaborators before submitting.

---

## Submission Checklist

- [ ] Dataset meets all five criteria from Step 3
- [ ] Data quality summary answers all four questions from Step 10 (written directly in `index.md`)
- [ ] Cleaning code runs without errors and shows before and after row counts
- [ ] Visualization 1 is complete, labeled, and has written context
- [ ] Visualization 2 is complete, labeled, and has written context
- [ ] The two visualizations use different chart types and answer different questions
- [ ] Reflection section answers all four questions
- [ ] Project builds with `npm run build` without errors
- [ ] Live Vercel URL loads and both charts render correctly
- [ ] Both links submitted on Canvas

---

## Quick Reference: Data Cleaning Patterns

| Problem | Detection | Fix |
|---|---|---|
| Missing values | `.filter(d => d.col == null)` | Remove row or impute with mean/median |
| Duplicates | `new Set()` comparison | Filter with a `seen` Set |
| Wrong type | `typeof d.col` | `Number()`, `new Date()`, `.toString()` |
| Inconsistent strings | `[...new Set()]` | `.trim().toLowerCase()` |
| Sentinel values | Check `min` and `max` | Filter to plausible range |
| Mixed units | Domain knowledge | Convert to one unit, document it |

---

## Quick Reference: Chart Type Decision Guide

| Question type | Best chart |
|---|---|
| How does X change over time? | Line chart |
| How do categories compare? | Bar chart (sorted) |
| What is the relationship between X and Y? | Scatter plot |
| Where is this happening geographically? | Choropleth map or dot map |
| How is this value distributed? | Histogram or box plot |
| What fraction of the whole is each part? | Stacked bar or treemap |
