# Personal Finance Tracker
Custom financial tracking and budgeting dashboard

## Background
I spent years of frustration using free and commercial personal finance apps: MS Money, Mint, Quicken, Monarch.  All of them had a lot of great features, but none of them were perfect and most of them couldn't reliably pull data from my financial institutions. So I decided to write my own. For now, this still has a lot of manual processes - but these early efforts are getting the core backend and dashboard in place.

:warning: Please note this repo does not yet contain the full suite of code, db schemas, etc. Maybe someday, but I never really intended this to be public; so the code is an embarrasing mess and contains a ton of personal banking info. At the very least, though, I want to get enough published to be an inspiration to other DIYers.

## What You'll Need
- PHP
- a database (I use MySQL)
- some third party libraries for styling, graphs, and charts
  - [jQuery](https://jquery.com/)
  - [DataTables](https://datatables.net/)
  - [canvasJS](https://canvasjs.com/)
  - [Handsontable](https://handsontable.com/)
- a lot of time and patience to customize to your needs

## My Approach
... in progress ...

- snapshot, totals
- display, layout, printing
- security

### Database
My primary financial instituions (bank and credit cards) thankfully support data export to CSV, so the first step is ingest that data into a single, unified table:
```
CREATE TABLE `exp_track` (
  `id` int NOT NULL AUTO_INCREMENT,
  `src` varchar(20) NOT NULL,
  `date` date NOT NULL,
  `descr` varchar(100) NOT NULL,
  `cat` enum('DINING','BILLS','GROCERIES','FUEL','INCOME','CHARITY','CCPAYMENTS','MORTGAGE','MEDICAL','SHOPPING','ENTERTAINMENT','TRAVEL','INTEREST','DOG','UNCAT') DEFAULT NULL,
  `amt` decimal(10,2) NOT NULL,
  `importdate` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `CAT_IDX` (`cat`),
  KEY `DATE_IDX` (`date`),
  FULLTEXT KEY `DESCR_IDX` (`descr`)
);
```
The `src` column (which I should make an enum) represents where the transaction is from.  I have values here like `checking`, `visa`, etc. The `cat` column (not all my own values shown) are transaction categories. Making that enum helps me enforce the data quality later when I run the categorizations on imported transactions.

For most of those `cat` categories, I have separate tables for each which I later use for each transaction. For example:
```
CREATE TABLE `cats_groceries` (
  `descr` varchar(100) NOT NULL
);
```
And that just has a list of substrings (in uppercase) that I will check for during import (e.g. 'GIANT FOOD','SAFEWAY','FOOD LION')  If a transaction description in `EXP_TRACK` contains one of those values, the transaction is categorized as 'GROCERIES'.

### Import Process
There is a separate screen to initiate the import process.  I take a CSV export from a given financial institution and upload it via a regular `multipart/form-data` FORM. That form also specifies which source the import is from, since each institution has a slightly different column order. Data is imported via MySQL `LOAD DATA` into the main `EXP_TRACK` table.

### Snapshot Totals and Balance Totals Trends
Independent of the data just imported, the top "Overview" section of the dashboard represents snapshots of my financial picture from various institutions. Figures like, "what was the balance on my checking account at the time of the most recent statement." I capture these amounts, categorized by asset or liability, and track them for trend analysis. Data is stored in the following table:
```
CREATE TABLE `snapshots` (
  `id` int NOT NULL AUTO_INCREMENT,
  `type` enum('assets','expenses') NOT NULL,
  `source` enum('checking','401k','mortgage','visa','529') NOT NULL,
  `date` date NOT NULL,
  `amount` mediumint NOT NULL,
  PRIMARY KEY (`id`)
);
```
Again, INSERTs to this table are still a very manual process.  Figures like a statement balance aren't included in transaction CSVs.

Once a have a full month of snapshot data, I run a series of queries to populate a `SNAPSHOT_TOTALS` table.  These monthly totals are then used to feed the "Balance Totals Trends" line graph.
```
CREATE TABLE `snapshot_totals` (
  `id` int NOT NULL AUTO_INCREMENT,
  `date` date NOT NULL,
  `amt_assets` mediumint NOT NULL,
  `amt_assets_filtered` mediumint NOT NULL,
  `amt_income_total` mediumint NOT NULL,
  `amt_expenses` mediumint NOT NULL,
  `amt_expenses_filtered` mediumint NOT NULL,
  `amt_expenses_total` mediumint NOT NULL,
  `amt_lily` mediumint NOT NULL,
  PRIMARY KEY (`id`)
);
```
More manual processes for now!  To populate this table, I run the following queries. First, this query gets the total assets or expenses from `SNAPSHOTS` for the previous month.  (I run this on the 1st.)
```
select sum(amount) as total
from (
 select a.source, a.amount as amount, a.date
     from snapshots a
     where a.type = 'expenses' and a.date = (select max(d.date) from snapshots d where d.source = a.source and d.date <= '2025-03-01') 
     group by a.source
) as results
```
For the example query above here, I would enter that result total into `SNAPSHOT_TOTALS.AMT_EXPENSES` where date = 2025-03-01. I use phpMyAdmin for a lot of this since the gui already has quick edit-in-place capabilities.

Next, I run an almost identical query as before, but filter out a few of the sources:
```
SELECT SUM(amount) AS total
FROM (
    SELECT a.source, a.amount AS amount, a.date
    FROM snapshots a
    WHERE a.type = 'assets' 
      AND a.source not in ('401k','mortgage')
      AND a.date =(SELECT MAX(d.date) FROM snapshots d WHERE d.source = a.source AND d.date <= '2025-03-01')
    GROUP BY a.source
) AS results
```
In doing so, I get visibility into a more granular financial picture without large items like my regular mortgage payments or my running 401k balance.  As before, that result total is inserted into the respective column in `SNAPSHOT_TOTALS` for filtered assets or expenses for the given date.

## Personal Limitations
- import workarounds due to hosting provider

## Screenshots
![main view](screenshots/main.png "main view")

## To Dos to Consider
- purge scheme (data over x years old)
- more/better automation
- improve txn categorization during import
