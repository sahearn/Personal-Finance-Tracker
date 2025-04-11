# Personal Finance Tracker
Custom financial tracking and budgeting dashboard

## Background
I spent years of frustration using free and commercial personal finance apps: MS Money, Mint, Quicken, Monarch.  All of them had a lot of great features, but none of them were perfect and most of them couldn't  reliably pull data from my financial institutions. So I decided to write my own.

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

- db tables
  - main, categories
- import
- snapshot, totals
- display, layout, printing
- security

### Database
My primary financial instituions (bank and credit cards) thankfully support data export to CSV, so the first step is ingest that data into a single, unified table:
```
CREATE TABLE `exp_track` (
  `id` int NOT NULL AUTO_INCREMENT,
  `src` varchar(20) COLLATE utf8mb4_general_ci NOT NULL,
  `date` date NOT NULL,
  `descr` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `cat` enum('DINING','BILLS','GROCERIES','FUEL','INCOME','CHARITY','CCPAYMENTS','MORTGAGE','MEDICAL','SHOPPING','ENTERTAINMENT','TRAVEL','INTEREST','DOG','UNCAT') CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `amt` decimal(10,2) NOT NULL,
  `importdate` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `CAT_IDX` (`cat`),
  KEY `DATE_IDX` (`date`),
  FULLTEXT KEY `DESCR_IDX` (`descr`)
) ENGINE=InnoDB AUTO_INCREMENT=8447 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```
The `src` column (which I should make an enum) represents where the transaction is from.  I have values here like `checking`, `visa`, etc. The `cat` column (not all my own values shown) are transaction categories. Making that enum helps me enforce the data quality later when I run the categorizations on imported transactions.

For most of those `cat` categories, I have separate tables for each which I later use for each transaction. For example:
```
CREATE TABLE `cats_groceries` (
  `descr` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```
And that just has a list of substrings (in uppercase) that I will check for during import (e.g. 'GIANT FOOD','SAFEWAY','FOOD LION')  If a transaction description in `EXP_TRACK` contains one of those values, the transaction is categorized as 'GROCERIES'.

### Import Process


## Personal Limitations
- import workarounds due to hosting provider

## Screenshots
![main view](screenshots/main.png "main view")

## To Dos to Consider
- purge scheme (data over x years old)
- more/better automation
- improve txn categorization during import
