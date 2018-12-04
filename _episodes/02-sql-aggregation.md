---
title: "SQL Aggregation"
teaching: 30
exercises: 5
questions:
- "How can I summarize my data by aggregating, filtering, or ordering query results?"
objectives:
- "Apply aggregation to group records in SQL."
- "Filter and order results of a query based on aggregate functions."
- "Save a query to make a new table."
- "Apply filters to find missing values in SQL."
keypoints:
- "Use the `GROUP BY` keyword to aggregate data."
- "Functions like `MIN`, `MAX`, `AVERAGE`, `SUM`, `COUNT`, etc. operate on aggregated data."
- "Use the `HAVING` keyword to filter on aggregate properties."
- "Use a `VIEW` to access the result of a query as though it was a new table."
---

## COUNT and GROUP BY

Aggregation allows us to combine results by grouping records based on value and
calculating combined values in groups.

Let’s go to the eebo table and find out how many individual titles there are.
Using the wildcard simply counts the number of records (rows):

    SELECT COUNT(*)
    FROM eebo;

We can also find out the total page length:

    SELECT COUNT(*), SUM(PageCount)
    FROM eebo;


There are many other aggregate functions included in SQL including
`MAX`, `MIN`, and `AVG`.

> ## Challenge
>
> Write a query that returns: total page length, average page length, and the min and max page lengths
> for all titles encoded over the duration of the project.
> Can you modify it so that it outputs these values only for page lengths between 1 and 10?
{: .challenge}

Now, let's see how many individuals were counted in each species. We do this
using a `GROUP BY` clause

    SELECT TCP, COUNT(*)
    FROM eebo
    GROUP BY Status;

`GROUP BY` tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give `GROUP BY` a comma separated list.

> ## Challenge
>
> Write queries that return:
>
> 1. How many groups of terms were created in each year
>    *   in total
>    *   per author
> 2. Average number of each term groupings in each year.
>
> Can you modify the above queries combining them into one?
{: .challenge}

## The `HAVING` keyword

In the previous lesson, we have seen the keywords `WHERE`, allowing to
filter the results according to some criteria. SQL offers a mechanism to
filter the results based on aggregate functions, through the `HAVING` keyword.

For example, we can adapt the last request we wrote to only return information
about page length with a count higher than 100:

    SELECT TCP, PageCount
    FROM eebo
    GROUP BY PageCount
    HAVING PageCount > 10;

The `HAVING` keyword works exactly like the `WHERE` keyword, but uses
aggregate functions instead of database fields.

If you use `AS` in your query to rename a column, `HAVING` can use this
information to make the query more readable. For example, in the above
query, we can call the `COUNT(pages)` by another name, like
`page`. This can be written this way:

    SELECT TCP, author, PageCount AS page
    FROM eebo
    GROUP BY TCP
    HAVING PageCount > 10;

Note that in both queries, `HAVING` comes *after* `GROUP BY`. One way to
think about this is: the data are retrieved (`SELECT`), can be filtered
(`WHERE`), then joined in groups (`GROUP BY`); finally, we only select some
of these groups (`HAVING`).

> ## Challenge
>
> Write a query that returns, from the `authors` table, the `eebo` IDs
> in each `authors`, only for the `authors` with more than 5 works.
{: .challenge}

## Ordering Aggregated Results

We can order the results of our aggregation by a specific column, including
the aggregated column.  Let’s count the number of individuals of each
species captured, ordered by the count:

    SELECT author, COUNT(*)
    FROM eebo
    GROUP BY author
    ORDER BY COUNT(author);

## Saving Queries for Future Use

It is not uncommon to repeat the same operation more than once, for example
for monitoring or reporting purposes. SQL comes with a very powerful mechanism
to do this: views. Views are a form of query that is saved in the database,
and can be used to look at, filter, and even update information. One way to
think of views is as a table, that can read, aggregate, and filter information
from several places before showing it to you.

Creating a view from a query requires to add `CREATE VIEW viewname AS`
before the query itself. For example, imagine that my project only covers
the data gathered of books published between 1642 - 1651.  That
query would look like:

    SELECT *
    FROM eebo
    WHERE (date > '1558' AND date < '1603');

But we don't want to have to type that every time we want to ask a
question about that particular subset of data.  Let's create a view:

    CREATE VIEW elizabethan AS
    SELECT *
    FROM eebo
    WHERE (date > '1558' AND date < '1603');

You can also add a view using *Create View* in the *View* menu and see the
results in the *Views* tab just like a table.

Now, we will be able to access these results with a much shorter notation:

    SELECT *
    FROM elizabethan;

There should only be 51 records.  If you look at the `PageCount` column, it's
easy to see what the average page length would be.  If we use SQL to find the
average page count of books that are available, SQL behaves like we would hope, 
ignoring the NULL values:

    SELECT AVG(PageCount)
    FROM elizabethan
    WHERE author != '';

But if we try to be extra clever, and find the average ourselves,
we might get tripped up:

    SELECT SUM(PageCount), COUNT(*), SUM(PageCount)/COUNT(*)
    FROM elizabethan
    WHERE author != '';

Here the `COUNT` command includes all 51 records (even those with NULL
values), but the `SUM` only includes the 49 records with data in the
`weight` field, giving us an incorrect average.  However,
our strategy *will* work if we modify the count command slightly:

    SELECT SUM(PageCount), COUNT(PageCount), SUM(PageCount)/COUNT(PageCount)
    FROM elizabethan
    WHERE author != '';

When we count the pages field specifically, SQL ignores the records with data
missing in that field.  So here is one example where NULLs can be tricky:
`COUNT(*)` and `COUNT(field)` can return different values.

Another case is when we use a "negative" query.  Let's count all the
non-free titles:

    SELECT COUNT(*)
    FROM elizabethan
    WHERE Author != '';

Now let's count all the titles wth no author listed:

    SELECT COUNT(*)
    FROM elizabethan
    WHERE Author != '';

But if we compare those two numbers with the total:

    SELECT COUNT(*)
    FROM elizabethan;

We'll see that they don't add up to the total!  That's because SQL
doesn't automatically include NULL values in a negative conditional
statement.  So if we are quering "not x", then SQL divides our data
into three categories: 'x', 'not NULL, not x' and NULL and
returns the 'not NULL, not x' group. Sometimes this may be what we want -
but sometimes we may want the missing values included as well!  In that
case, we'd need to change our query to:

    SELECT COUNT(*)
    FROM eebo
    WHERE author != '' OR author IS NULL;

There is one more subtlety we need to be aware of.
Suppose we run this query:

    SELECT COUNT(*), PageCount
    FROM elizabethen;

