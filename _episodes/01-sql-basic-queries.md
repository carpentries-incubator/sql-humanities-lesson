---
title: "Basic Queries"
teaching: 30
exercises: 5
questions:
- "How do I write a basic query in SQL?"
objectives:
- "Write and build queries."
- "Filter data given various criteria."
- "Sort the results of a query."
keypoints:
- "It is useful to apply conventions when writing SQL queries to aid readability."
- "Use logical connectors such as AND or OR to create more complex queries."
- "Calculations using mathematical symbols can also be performed on SQL queries."
- "Adding comments in SQL helps keep complex queries understandable."
---

## Writing my first query

Let's start by using the **eebo** table. Here we have data on every
title that is included in the catalogue.

Let’s write an SQL query that selects only the year column from the
surveys table. SQL queries can be written in the box located under 
the "Execute SQL" tab. Click 'Run SQL' to execute the query in the box.

    SELECT Title
    FROM eebo;

We have capitalized the words SELECT and FROM because they are SQL keywords.
SQL is case insensitive, but it helps for readability, and is good style.

If we want more information, we can just add a new column to the list of fields,
right after SELECT:

    SELECT Title, TCP
    FROM eebo;

Or we can select all of the columns in a table using the wildcard *

    SELECT *
    FROM eebo;

### Limiting results

Sometimes you don't want to see all the results you just want to get a sense of
of what's being returned. In that case you can use the LIMIT command. In particular
you would want to do this if you were working with large databases.

    SELECT *
    FROM eebo
    LIMIT 10; 

### Unique values

If we want only the unique values so that we can quickly see what authors have
been cataloged we use `DISTINCT` 

    SELECT DISTINCT Date
    FROM eebo;

If we select more than one column, then the distinct pairs of values are
returned

    SELECT DISTINCT Date, Title
    FROM eebo;

### Calculated values

We can also do calculations with the values in a query.
For example, if we wanted to look at the number of pages associated with 
different dates, but we needed to know how many tens of pages we would use

    SELECT Date, PageCount/10
    FROM eebo;

When we run the query, the expression `pages / 10` is evaluated for each
row and appended to that row, in a new column. If we used the `INTEGER` data type
for the pages field then integer division would have been done, to obtain the
correct results in that case divide by `10.0`. Expressions can use any fields,
any arithmetic operators (`+`, `-`, `*`, and `/`) and a variety of built-in
functions. For example, we could round the values to make them easier to read.

    SELECT TCP, title, ROUND(pages / 10, 2)
    FROM eebo;

> ## Challenge
>
> - Write a query that returns the year, EEBO and page length
{: .challenge}

## Filtering

Databases can also filter data – selecting only the data meeting certain
criteria.  For example, let’s say we only want data for the titles that
have a free status. We need to add a `WHERE` clause to our query:

    SELECT *
    FROM eebo
    WHERE status='Free';

We can do the same thing with numbers.
Here, we only want the data since 1640:

    SELECT * 
    FROM eebo
    WHERE date >= '1600';

If we used the `TEXT` data type for the year the `WHERE` clause should
be `Date >= '1600'`. We can use more sophisticated conditions by combining tests
with `AND` and `OR`.  For example, suppose we want the data on *Free* status
starting in the year 1600:

    SELECT *
    FROM catalogue
    WHERE (date >= '1600') AND (status = 'Free');

Note that the parentheses are not needed, but again, they help with
readability.  They also ensure that the computer combines `AND` and `OR`
in the way that we intend.

> ## Challenge
>
> - Produce a table listing the data for all titles in the catalogue 
> with a page length more than 75, telling us the date, TCP id code, and page. 
{: .challenge}

## Building more complex queries

Now, lets combine the above queries to get data for 2 authors from
the year 1580 on.  This time, let’s use IN as one way to make the query easier
to understand.  It is equivalent to saying `WHERE (author = 'Aylett, Robert, 1583-1655?') OR (author
= 'Bacon, Francis, 1561-1626.'), but reads more neatly:

    SELECT *
    FROM eebo
    WHERE (date >= '1580') AND (author IN ('Aylett, Robert, 1583-1655?', 'Bacon, Francis, 1561-1626.'));

We started with something simple, then added more clauses one by one, testing
their effects as we went along.  For complex queries, this is a good strategy,
to make sure you are getting what you want.  Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.

When the queries become more complex, it can be useful to add comments. In SQL,
comments are started by `--`, and end at the end of the line. For example, a
commented version of the above query can be written as:

    -- Get post 1580 data on authors
    -- These are in the catalogue table, and we are interested in all columns
    SELECT * FROM eebo
    -- Sampling year is in the column `Date`, and we want to include after 1580
    WHERE (date >= '1580')
    -- Author names
    AND (author IN ('Aylett, Robert, 1583-1655?', 'Bacon, Francis, 1561-1626.'));

Although SQL queries often read like plain English, it is *always* useful to add
comments; this is especially true of more complex queries.

## Sorting

We can also sort the results of our queries by using `ORDER BY`.
For simplicity, let’s go back to the **catalogue** table and order by publication date.

First, let's look at what's in the **catalogue** table. It's a table of the eebo catalogue id and the 
information for each id.

    SELECT *
    FROM eebo;

Now let's order it by date.

    SELECT *
    FROM eebo
    ORDER BY date ASC;

The keyword `ASC` tells us to order it in Ascending order.
We could alternately use `DESC` to get descending order.

    SELECT *
    FROM eebo
    ORDER BY date DESC;

`ASC` is the default.

We can also sort on several fields at once.
To truly be alphabetical, we might want to order by date then author.

    SELECT *
    FROM eebo
    ORDER BY Date ASC, Author ASC;

> ## Challenge
>
> - Write a query that returns year, TCP id, and page from
> the catalogue table, sorted with the largest page lengths at the top.
{: .challenge}

## Order of execution

Another note for ordering. We don’t actually have to display a column to sort by
it.  For example, let’s say we want to order an author by the EEBO index, but
we only want to see genus and species.

    SELECT Title, Terms
    FROM eebo
    WHERE author = 'Bacon, Francis, 1561-1626.'
    ORDER BY TCP ASC;

We can do this because sorting occurs earlier in the computational pipeline than
field selection.

The computer is basically doing this:

1. Filtering rows according to WHERE
2. Sorting results according to ORDER BY
3. Displaying requested columns or expressions.

Clauses are written in a fixed order: `SELECT`, `FROM`, `WHERE`, then `ORDER
BY`. It is possible to write a query as a single line, but for readability,
we recommend to put each clause on its own line.

> ## Challenge
>
> - Let's try to combine what we've learned so far in a single
> query.  Using the catalogue table write a query to display the title,
> terms field and the page length (rounded to two decimal places), for
> titles published in 1550, ordered alphabetically by the author.
> - Write the query as a single line, then put each clause on its own line, and
> see how more legible the query becomes!
{: .challenge}

