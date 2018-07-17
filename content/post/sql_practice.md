---
author: "Donghua (Lyla) CAI"
date: 2018-04-15
title: "SQL Practices from Mode Analytics"
tags: [
    "Machine Learning",
	"SQL"
]
categories: [
    "SQL"
]
math: true
---
Practice from [modeanalytics.com](https://community.modeanalytics.com/sql/). This is a great tutorial of SQL. **Highly recommend it!** 

# 1. Intermediate SQL

## 1.1 Simple filter with `WHERE`

```SQL
/*
Table 1: benn.college_football_players

Write a query that selects the school name, player name, 
position, and weight for every player in Georgia, ordered 
by weight (heaviest to lightest). Be sure to make an alias 
for the table, and to reference all column names in 
relation to the alias.*/

SELECT players.school_name,
       players.player_name,
       players.position,
       players.weight
  FROM benn.college_football_players players
 WHERE players.state = 'GA'
 ORDER BY players.weight DESC
```

## 1.2 LEFT JOIN two tables

```SQL
/*
Table 1: tutorial.crunchbase_companies (permalink, state_code)
Table 2: tutorial.crunchbase_acquisitions (company_permalink)

Count the number of unique companies (don't double-count companies)
and unique acquired companies by state. Do not include results for
which there is no state data, and order by the number of acquired 
companies from highest to lowest.
*/

SELECT companies.state_code,
       COUNT(DISTINCT companies.permalink) AS unique_companies,
       COUNT(DISTINCT acquisitions.company_permalink) AS unique_companies_acquired
  FROM tutorial.crunchbase_companies companies
  LEFT JOIN tutorial.crunchbase_acquisitions acquisitions
    ON companies.permalink = acquisitions.company_permalink
 WHERE companies.state_code IS NOT NULL
 GROUP BY 1
 ORDER BY 3 DESC
```

## 1.3 JOINs using WHERE and ON

Some differences between codes below. The first one uses two clauses with `ON` using `AND`. The second one uses a `WHERE` clause. In the first case, **JOIN** happens after filtering; while in the second case, **JOIN** occurs before filtering.

```SQL
SELECT companies.permalink AS companies_permalink,
       companies.name AS companies_name,
       acquisitions.company_permalink AS acquisitions_permalink,
       acquisitions.acquired_at AS acquired_date
  FROM tutorial.crunchbase_companies companies
  LEFT JOIN tutorial.crunchbase_acquisitions acquisitions
    ON companies.permalink = acquisitions.company_permalink
   AND acquisitions.company_permalink != '/company/1000memories'
 ORDER BY 1

```

```SQL
SELECT companies.permalink AS companies_permalink,
       companies.name AS companies_name,
       acquisitions.company_permalink AS acquisitions_permalink,
       acquisitions.acquired_at AS acquired_date
  FROM tutorial.crunchbase_companies companies
  LEFT JOIN tutorial.crunchbase_acquisitions acquisitions
    ON companies.permalink = acquisitions.company_permalink
 WHERE acquisitions.company_permalink != '/company/1000memories'
    OR acquisitions.company_permalink IS NULL
 ORDER BY 1
```

## 1.4 LEFT JOIN
```SQL
/*
Table 1: tutorial.crunchbase_investments (company_permalink, investor_permalink)
Table 2: tutorial.crunchbase_companies (permalink, name, status, state_code)

Write a query that shows a company's name, "status" (found in
the Companies table), and the number of unique investors in 
that company. Order by the number of investors from most to 
fewest. Limit to only companies in the state of New York.
*/

SELECT company.name, 
       company.status, 
       COUNT(DISTINCT invest.investor_permalink) as unique_investor_count
FROM tutorial.crunchbase_companies company
LEFT JOIN tutorial.crunchbase_investments invest
ON company.permalink = invest.company_permalink
WHERE company.state_code = 'NY'
GROUP BY 1,2
ORDER BY 3 DESC

```

```SQL
/*
Table 1: tutorial.crunchbase_investments (company_permalink, investor_permalink, investor_name)
Table 2: tutorial.crunchbase_companies (permalink, name, status, state_code)

Write a query that lists investors based on the number of 
companies in which they are invested. Include a row for 
companies with no investor, and order from most companies 
to least.
*/

SELECT CASE WHEN invest.investor_name IS NULL THEN 'No_Investors'
            ELSE invest.investor_name END AS investor,
       COUNT(DISTINCT company.permalink) AS number_invested_companies
FROM tutorial.crunchbase_companies company
LEFT JOIN tutorial.crunchbase_investments invest
ON company.permalink = invest.company_permalink
GROUP BY 1
ORDER BY 2 DESC
```

## 1.5 FULL JOIN
```SQL
/*
Table1: tutorial.crunchbase_companies (permalink)
Table2: tutorial.crunchbase_investments_part1 (company_permalink)

Write a query that joins tutorial.crunchbase_companies and 
tutorial.crunchbase_investments_part1 using a FULL JOIN. 
Count up the number of rows that are matched/unmatched as
in the example above.
*/
    
SELECT COUNT(CASE WHEN companies.permalink IS NOT NULL AND investments.company_permalink IS NULL
                  THEN companies.permalink ELSE NULL END) AS companies_only,
       COUNT(CASE WHEN companies.permalink IS NOT NULL AND investments.company_permalink IS NOT NULL
                  THEN companies.permalink ELSE NULL END) AS both_tables,
       COUNT(CASE WHEN companies.permalink IS NULL AND investments.company_permalink IS NOT NULL
                  THEN investments.company_permalink ELSE NULL END) AS investments_only
FROM tutorial.crunchbase_companies companies
FULL JOIN tutorial.crunchbase_investments_part1 as investments
ON companies.permalink = investments.company_permalink
```

## 1.6 UNION / UNION ALL

It is used to append a table. Rules: 1) Both tables must have the same number of columns, typically same names of columns too. 2) The columns must have the same data types in the same order as the first table. 

```SQL
/*
Table1: tutorial.crunchbase_investments_part1 (company_permalink, company_name, investor_name)
Table2: tutorial.crunchbase_investments_part2 (company_permalink, company_name, investor_name)

Write a query that appends the two crunchbase_investments datasets 
above (including duplicate values). Filter the first dataset to 
only companies with names that start with the letter "T", and filter
the second to companies with names starting with "M" (both not 
case-sensitive). Only include the company_permalink, company_name,
and investor_name columns.
*/

(SELECT company_permalink, company_name, investor_name
FROM tutorial.crunchbase_investments_part1
WHERE company_name ILIKE 'T%')
UNION ALL
(SELECT company_permalink, company_name, investor_name
FROM tutorial.crunchbase_investments_part2
WHERE company_name ILIKE 'M%')
```

## 1.7 Self JOIN

For example, let’s say you wanted to identify companies that received an investment from Great Britain following an investment from Japan.

```SQL
SELECT DISTINCT japan_investments.company_name,
       japan_investments.company_permalink
  FROM tutorial.crunchbase_investments_part1 japan_investments
  JOIN tutorial.crunchbase_investments_part1 gb_investments
    ON japan_investments.company_name = gb_investments.company_name
   AND gb_investments.investor_country_code = 'GBR'
   AND gb_investments.funded_at > japan_investments.funded_at
 WHERE japan_investments.investor_country_code = 'JPN'
 ORDER BY 1
```

# 2. Advanced SQL

## 2.1 SQL Data Types

Supported [data types by SQL](https://www.w3schools.com/sql/sql_datatypes.asp). Two ways to convert data types: 1). `CAST(column_name AS integer)`; 2) `column_name::integer`

## 2.2 SQL Date Format

Two types of date: `date` and `timestamp` (timestamp havs additional precision hours,minutes,seconds). Useful data type for date format: `INTERVAL`, for example, `INTERVAL '3 years'`.

```SQL
/*
Table1: tutorial.crunchbase_companies_clean_date  (permalink, founded_at_clean, category_code)
Table2: tutorial.crunchbase_acquisitions_clean_date  (company_permalink, acquired_at_cleaned)

Write a query that counts the number of companies acquired
within 3 years, 5 years, and 10 years of being founded 
(in 3 separate columns). Include a column for total companies
acquired as well. Group by category and limit to only rows 
with a founding date.
*/

SELECT companies.category_code,
       COUNT(CASE WHEN (acquisitions.acquired_at_cleaned -
         companies.founded_at_clean::timestamp) <= INTERVAL '3 years' THEN 'YES'
         ELSE NULL END) AS less3y,
       COUNT(CASE WHEN (acquisitions.acquired_at_cleaned -
         companies.founded_at_clean::timestamp) <= INTERVAL '5 years' THEN 'YES'
         ELSE NULL END) AS less5y,
       COUNT(CASE WHEN (acquisitions.acquired_at_cleaned -
         companies.founded_at_clean::timestamp) <= INTERVAL '10 years' THEN 'YES'
         ELSE NULL END) AS less10y, 
       COUNT(companies.permalink) AS total
FROM tutorial.crunchbase_companies_clean_date companies
JOIN tutorial.crunchbase_acquisitions_clean_date acquisitions
ON companies.permalink = acquisitions.company_permalink
WHERE founded_at_clean IS NOT NULL
GROUP BY 1
ORDER BY 5 DESC
```

## 2.3 Using SQL String Functions to Clean Data

**2.3.A.** `LEFT` to extract a substring starting from left. `LEFT(string, number of characters)`. `Right`, similar as `LEFT`. `LENGTH` returns the length of a string. `SUBSTR` to extract a substring from the middle of a string. Example: `SUBSTR(string, starting character position, # of characters)`.

```SQL
SELECT id,
       date,
	   SUBSTR(date, 4, 2) AS day
FROM crime_data_table
```

**2.3.B.** `TRIM` is used to remove characters from the beginning and end of a string. `TRIM` takes 3 parameters: 1) location trim string, option: `BOTH/LEADING/TRAILIN`. 2) removed characters, default value is space, this function will remove characters in the string which are in removed characters. 3) specify the string you want to clean. 

```SQL
TRIM(BOTH '()' FROM location)
```

**2.3.C.** `POSITION` returns the location (count from the left) of a substring first which first appears in a string. Example: `POSITION('A' IN descript)`. Another function returns the same output is `STRPOS`.

**2.3.D.** `CONCAT` to combine strings. Example, 

```SQL
SELECT CONCAT('(', lat, ', ', lon, ')') AS concat_location,
       location
FROM crime_data_table
```

Which is equal to the below using `||`.
```SQL
SELECT '(' || lat || ', ' || lon || ')' AS concat_location,
       location
FROM crime_data_table
```

**2.3.E.** `UPPER` and `LOWER` to change case of string.

**2.3.F.** `AT TIME ZONE` can make a time appear in a different time zone. Example, `SELECT CURRENT_TIME AT TIME ZONE 'PST' AS time_pst`.

**2.3.G.** `COALESCE` to replace null values with another string. `SELECT COALESCE(descipt, 'No Description') FROM table`

## 2.4 Sub-query

An example below (it does not need subquery for this question, just to demonstrate subquery):

```SQL
/*
Table: tutorial.sf_crime_incidents_2014_01 (descript, resolution)

Write a query that selects all Warrant Arrests from
the tutorial.sf_crime_incidents_2014_01 dataset, 
then wrap it in an outer query that only displays 
unresolved incidents.
*/

SELECT sub.*
FROM ( SELECT *
       FROM tutorial.sf_crime_incidents_2014_01
       WHERE descript = 'WARRANT ARREST'
       ) sub
WHERE sub.resolution = 'NONE'

```

**2.4.A.** Using subQueries to aggregate 

```SQL
/*
Get average incidents on Friday in December
*/

SELECT LEFT(sub.date, 2) AS cleaned_month,
       sub.day_of_week,
       AVG(sub.incidents) AS average_incidents
  FROM (
        SELECT day_of_week,
               date,
               COUNT(incidnt_num) AS incidents
          FROM tutorial.sf_crime_incidents_2014_01
         GROUP BY 1,2
       ) sub
 GROUP BY 1,2
 ORDER BY 1,2
```

```SQL
/*
Table: tutorial.sf_crime_incidents_cleandate (date, category, incidnt_num)

Write a query that displays the average number of monthly
incidents for each category. Hint: use 
tutorial.sf_crime_incidents_cleandate to make your life a 
little easier.
*/
SELECT sub.category, AVG(sub.inc_count)
FROM(
SELECT LEFT(date, 2) as month,
       category,
       COUNT(incidnt_num) as inc_count
FROM tutorial.sf_crime_incidents_cleandate
GROUP BY 1, 2) sub
GROUP BY 1
```

**2.4.B.** Subqueries in conditinal logic

```SQL
/*
Table: tutorial.sf_crime_incidents_2014_01
Select crime that happened in the earliest day in the dataset.
*/

SELECT *
FROM tutorial.sf_crime_incidents_2014_01 
WHERE Date = (SELECT MIN(date)
                 FROM tutorial.sf_crime_incidents_2014_01
              )
```

**Example:** Select crimes that happened in the ealiest 5 days. **Notes:** subquery should not have an alias with `IN` conditional logic. Because the result of this kind of subquery is treated as set of values not table. 

```SQL
SELECT *
  FROM tutorial.sf_crime_incidents_2014_01
 WHERE Date IN (SELECT date
                 FROM tutorial.sf_crime_incidents_2014_01
                ORDER BY date
                LIMIT 5
              )
```

**2.4.C.** Joining subqueries

```SQL
/*
Table: tutorial.sf_crime_incidents_2014_01 (category)
Write a query that displays all rows from the three 
categories with the fewest incidents reported.
*/

SELECT crime.*, sub.inc_num_that_day
FROM tutorial.sf_crime_incidents_2014_01 crime 
JOIN
(SELECT category, 
       COUNT(*) as inc_num_that_day
FROM tutorial.sf_crime_incidents_2014_01
GROUP BY 1
ORDER BY inc_num_that_day
LIMIT 3) sub
ON crime.category = sub.category
```

**Task:** aggregate all of the companies receiving investment and 
companies acquired each month. Below is a neat solution but it takes a long time to run because two `DISTINCT` run on large table.

```SQL
/* Slow way
*/
SELECT COALESCE(acquisitions.acquired_month, investments.funded_month) AS month,
       COUNT(DISTINCT acquisitions.company_permalink) AS companies_acquired,
       COUNT(DISTINCT investments.company_permalink) AS investments
  FROM tutorial.crunchbase_acquisitions acquisitions
  FULL JOIN tutorial.crunchbase_investments investments
    ON acquisitions.acquired_month = investments.funded_month
 GROUP BY 1

```

Another long but more efficient solution. Aggregate the two tables separately, then join them together. `DISTINCT` runs on smaller tables in this way.
```SQL
SELECT COALESCE(acquisitions.month, investments.month) AS month,
       acquisitions.companies_acquired,
       investments.companies_rec_investment
  FROM (
        SELECT acquired_month AS month,
               COUNT(DISTINCT company_permalink) AS companies_acquired
          FROM tutorial.crunchbase_acquisitions
         GROUP BY 1
       ) acquisitions

  FULL JOIN (
        SELECT funded_month AS month,
               COUNT(DISTINCT company_permalink) AS companies_rec_investment
          FROM tutorial.crunchbase_investments
         GROUP BY 1
       )investments

    ON acquisitions.month = investments.month
 ORDER BY 1 DESC
```

```SQL
/*
Table1: tutorial.crunchbase_companies (founded_qusrter, permalink)
Table2: tutorial.crunchbase_acquisitions (acquired_quarter, company_permalink)

Write a query that counts the number of companies founded and
acquired by quarter starting in Q1 2012. Create the aggregations
in two separate queries, then join them.
*/

SELECT COALESCE(companies.quarter, acquisitions.quarter) AS quarter,
           companies.companies_founded,
           acquisitions.companies_acquired
      FROM (
            SELECT founded_quarter AS quarter,
                   COUNT(permalink) AS companies_founded
              FROM tutorial.crunchbase_companies
             WHERE founded_year >= 2012
             GROUP BY 1
           ) companies
      
      LEFT JOIN (
            SELECT acquired_quarter AS quarter,
                   COUNT(DISTINCT company_permalink) AS companies_acquired
              FROM tutorial.crunchbase_acquisitions
             WHERE acquired_year >= 2012
             GROUP BY 1
           ) acquisitions
        
        ON companies.quarter = acquisitions.quarter
     ORDER BY 1
```

**2.4.D.** Subqueries and Union

```SQL
/*
Table1:  tutorial.crunchbase_investments_part1 (company_permalink)
Table2:  tutorial.crunchbase_investments_part2 (company_permalink)
Table3:  tutorial.crunchbase_companies (permalink, status)
Write a query that does the same thing as in the previous problem,
except only for companies that are still operating. Hint: operating
status is in tutorial.crunchbase_companies.
*/

SELECT investments.investor_name,
       COUNT(investments.*) AS investments
  FROM tutorial.crunchbase_companies companies
  JOIN (
        SELECT *
          FROM tutorial.crunchbase_investments_part1
         
         UNION ALL
        
         SELECT *
           FROM tutorial.crunchbase_investments_part2
       ) investments
    ON investments.company_permalink = companies.permalink
 WHERE companies.status = 'operating'
 GROUP BY 1
 ORDER BY 2 DESC
```

## 2.5 SQL Window Functions

**2.5.A.** ROW\_NUMBER()

**2.5.B.** RANK() and DENSE_RANK()

```SQL
/*
Table: tutorial.dc_bikeshare_q1_2012 (start_terminal, start_time, duration_seconds)
 
Write a query that shows the 5 longest rides from each starting 
terminal, ordered by terminal, and longest to shortest rides 
within each terminal. Limit to rides that occurred before Jan. 8, 2012.
*/

SELECT *
  FROM (
        SELECT start_terminal,
               start_time,
               duration_seconds AS trip_time,
               RANK() OVER (PARTITION BY start_terminal 
                    ORDER BY duration_seconds DESC) AS rank
          FROM tutorial.dc_bikeshare_q1_2012
         WHERE start_time < '2012-01-08'
               ) sub
 WHERE sub.rank <= 5
```

**2.5.C.** NTILE

`NTILE` to identify what percentile (or quartile) a given row falls into. Syntax is `NTILE(# of buckets)`. 

```SQL
SELECT start_terminal,
       duration_seconds,
       NTILE(4) OVER
         (PARTITION BY start_terminal ORDER BY duration_seconds)
          AS quartile,
       NTILE(5) OVER
         (PARTITION BY start_terminal ORDER BY duration_seconds)
         AS quintile,
       NTILE(100) OVER
         (PARTITION BY start_terminal ORDER BY duration_seconds)
         AS percentile
  FROM tutorial.dc_bikeshare_q1_2012
 WHERE start_time < '2012-01-08'
 ORDER BY start_terminal, duration_seconds
```

Define a `WINDOW` to simplify the query. `WINDOW` clause should always come after the `WHERE` clause. 

```SQL
SELECT start_terminal,
       duration_seconds,
       NTILE(4) OVER ntile_window AS quartile,
       NTILE(5) OVER ntile_window AS quintile,
       NTILE(100) OVER ntile_window AS percentile
  FROM tutorial.dc_bikeshare_q1_2012
 WHERE start_time < '2012-01-08'
WINDOW ntile_window AS
         (PARTITION BY start_terminal ORDER BY duration_seconds)
 ORDER BY start_terminal, duration_seconds
```

```SQL
/*
Table: tutorial.dc_bikeshare_q1_2012 (duration_seconds, start_time)

Write a query that shows only the duration of the trip and the percentile
into which that duration falls (across the entire dataset—not partitioned 
by terminal).
*/

SELECT duration_seconds,
       NTILE(100) OVER (ORDER BY duration_seconds)
         AS percentile
  FROM tutorial.dc_bikeshare_q1_2012
 WHERE start_time < '2012-01-08'
 ORDER BY 1 DESC
```

**2.5.D.** LAG and LEAD

Use `LAG` and `LEAD` to compare rows to preceding or following rows. 

## 2.6 [Pivoting Data by SQL](https://community.modeanalytics.com/sql/tutorial/sql-pivot-table/)

**2.6.A**  Pivoting columns to rows

$1^{st}$. create a table that lists all the columns from the original table. 

```SQL
SELECT year
  FROM (VALUES (2000),(2001),(2002),(2003),(2004),(2005),(2006),
               (2007),(2008),(2009),(2010),(2011),(2012)) v(year)
```

$2^{nd}$. Join it with `worldwide_earthquakes` table to create an expanded view:

```SQL
SELECT years.*,
       earthquakes.*
  FROM tutorial.worldwide_earthquakes earthquakes
 CROSS JOIN (
       SELECT year
         FROM (VALUES (2000),(2001),(2002),(2003),(2004),(2005),(2006),
                      (2007),(2008),(2009),(2010),(2011),(2012)) v(year)
       ) years
```

$3^{rd}$. Pull the correct column using `CASE`.

```SQL
SELECT years.*,
       earthquakes.magnitude,
       CASE year
         WHEN 2000 THEN year_2000
         WHEN 2001 THEN year_2001
         WHEN 2002 THEN year_2002
         WHEN 2003 THEN year_2003
         WHEN 2004 THEN year_2004
         WHEN 2005 THEN year_2005
         WHEN 2006 THEN year_2006
         WHEN 2007 THEN year_2007
         WHEN 2008 THEN year_2008
         WHEN 2009 THEN year_2009
         WHEN 2010 THEN year_2010
         WHEN 2011 THEN year_2011
         WHEN 2012 THEN year_2012
         ELSE NULL END
         AS number_of_earthquakes
  FROM tutorial.worldwide_earthquakes earthquakes
 CROSS JOIN (
       SELECT year
         FROM (VALUES (2000),(2001),(2002),(2003),(2004),(2005),(2006),
                      (2007),(2008),(2009),(2010),(2011),(2012)) v(year)
       ) years
```
