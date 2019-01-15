# mysql-standards
Coding standards and guide for developing apps using MySQL


### Table of contents
1.  [ General ](#general)  
2.  [ SELECT](#select)  
	2.1.  [ Good ](#select-good)   
	2.2.  [ Bad ](#select-bad)   
3.  [ FROM](#from)  
	3.1.  [ Good ](#from-good)   
	3.2.  [ Bad ](#from-bad)   
4.  [ JOIN](#join)  
	4.1.  [ Good ](#join-good)   
	4.2.  [ Bad ](#join-bad)   
	4.3.  [ Good ](#join-good-1)   
	4.4.  [ Bad ](#join-bad-1)   
5.  [ WHERE](#where)  
6.  [ CASE](#case)  
7.  [ Common Table Expressions (CTEs)](#cte)  
	3.1.  [ Good ](#cte-good)   
	3.2.  [ Bad ](#cte-bad)   

<a id="general"></a>
# General

-   No tabs. 4 spaces per indent.
    
-   No trailing whitespace.
    
-   Always capitalize SQL keywords (e.g.,  `SELECT`  or  `AS`)
    
-   Variable names should be **underscore** separated:
    
    **GOOD**:  `SELECT COUNT(*) AS backers_count`
    
    **BAD**:  `SELECT COUNT(*) AS backersCount`
    
-   Comments should go near the top of your query, or at least near the closest  `SELECT`
    
-   Try to only comment on things that aren't obvious about the query (e.g., why a particular ID is hardcoded, etc.)
    
-   Don't use single letter variable names be as descriptive as possible given the context:
    
    **GOOD**:  `SELECT ksr.backings AS backings_with_creators`
    
    **BAD**:  `SELECT ksr.backings AS b`
    
-   Use  [Common Table Expressions](http://www.postgresql.org/docs/8.4/static/queries-with.html)  (CTEs) early and often, and name them well.
    
-   `HAVING`  isn't supported in Redshift, so use CTEs instead. If you don't know what this means, ask a friendly Data Team member.
    

<a id="selects"></a>
# SELECT

Align all columns to the first column on their own line:

    SELECT
        projects.name,
        users.email,
        projects.country,
        COUNT(backings.id) AS backings_count
    FROM ...

  

`SELECT`  goes on its own line:

    SELECT
        name,
        ...

  

Always rename aggregates and function-wrapped columns:

    SELECT
        name,
        SUM(amount) AS sum_amount
    FROM ...

  

Always rename all columns when selecting with table aliases:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.backings AS backings
    INNER JOIN ksr.projects AS projects ON ...

  

Always use  `AS`  to rename columns:

<a id="select-good"></a>
## GOOD:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    ...

  
<a name="select_bad"></a>
## BAD:

    SELECT
        projects.name project_name,
        COUNT(backings.id) backings_count
    ...

  

Long Window functions should be split across multiple lines: one for the  `PARTITION`,  `ORDER`  and frame clauses, aligned to the  `PARTITION`  keyword. Partition keys should be one-per-line, aligned to the first, with aligned commas. Order (`ASC`,  `DESC`) should always be explicit. All window functions should be aliased.

SUM(1) OVER (PARTITION BY category_id,
                          year
             ORDER BY pledged DESC
             ROWS UNBOUNDED PRECEDING) AS category_year

  
<a id="from"></a>
# FROM

Only one table should be in the  `FROM`. Never use  `FROM`-joins:

<a id="from-good"></a>
## GOOD:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects
    INNER JOIN ksr.backings AS backings ON backings.project_id = projects.id
    ...

  
<a id="from-bad"></a>
## BAD:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects, ksr.backings AS backings
    WHERE
        backings.project_id = projects.id
    ...

  
<a id="join"></a>
# JOIN

Explicitly use  `INNER JOIN`  not just  `JOIN`, making multiple lines of  `INNER JOIN`s easier to scan:

<a id="join-good"></a>
## GOOD:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects
    INNER JOIN ksr.backings AS backings ON ...
    INNER JOIN ...
    LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
    LEFT JOIN ...

<a id="join-bad"></a>
## BAD:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects
    JOIN ksr.backings AS backings ON ...
    LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
    LEFT JOIN ...

  

Additional filters in the  `INNER JOIN`  go on new indented lines:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects
    INNER JOIN ksr.backings AS backings ON projects.id = backings.project_id
        AND backings.project_country != 'US'
    ...

  

The  `ON`  keyword and condition goes on the  `INNER JOIN`  line:

    SELECT
        projects.name AS project_name,
        COUNT(backings.id) AS backings_count
    FROM ksr.projects AS projects
    INNER JOIN ksr.backings AS backings ON projects.id = backings.project_id
    ...

  

Begin with  `INNER JOIN`s and then list  `LEFT JOIN`s, order them semantically, and do not intermingle  `LEFT JOIN`s with  `INNER JOIN`s unless necessary:

<a id="join-good-1"></a>
## GOOD:

    INNER JOIN ksr.backings AS backings ON ...
    INNER JOIN ksr.users AS users ON ...
    INNER JOIN ksr.locations AS locations ON ...
    LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
    LEFT JOIN ...

<a id="join-bad-1"></a>
## BAD:

    LEFT JOIN ksr.backer_rewards AS backer_rewards ON backings
    INNER JOIN ksr.users AS users ON ...
    LEFT JOIN ...
    INNER JOIN ksr.locations AS locations ON ...

# WHERE

Multiple  `WHERE`  clauses should go on different lines and begin with the SQL operator:

    SELECT
        name,
        goal
    FROM ksr.projects AS projects
    WHERE
        country = 'US'
        AND deadline >= '2015-01-01'
	...


<a id="case"></a>
# CASE

`CASE`  statements aren't always easy to format but try to align  `WHEN`,  `THEN`, and  `ELSE`  together inside  `CASE`  and  `END`:

    CASE WHEN category = 'Art'
        THEN backer_id
        ELSE NULL
    END

  

  
<a id="cte"></a>
# Common Table Expressions (CTEs)

The body of a CTE must be one indent further than the  `WITH`  keyword. Open them at the end of a line and close them on a new line:

    WITH backings_per_category AS (
        SELECT
            category_id,
            deadline,
        ...
    )

  

Multiple CTEs should be formatted accordingly:

    WITH backings_per_category AS (
        SELECT
            ...
    ), backers AS (
        SELECT
            ...
    ), backers_and_creators AS (
        ...
    )
    SELECT * FROM backers;

  

If possible,  `JOIN`  CTEs inside subsequent CTEs, not in the main clause:

<a id="cte-good"></a>
## GOOD:

    WITH backings_per_category AS (
        SELECT
            ...
    ), backers AS (
        SELECT
            backer_id,
            COUNT(backings_per_category.id) AS projects_backed_per_category
        INNER JOIN ksr.users AS users ON users.id = backings_per_category.backer_id
    ), backers_and_creators AS (
        ...
    )
    SELECT * FROM backers_and_creators;

  
<a id="cte-bad"></a>
## BAD:

    WITH backings_per_category AS (
        SELECT
            ...
    ), backers AS (
        SELECT
            backer_id,
            COUNT(backings_per_category.id) AS projects_backed_per_category
    ), backers_and_creators AS (
        ...
    )
    SELECT * FROM backers_and_creators
    INNER JOIN backers ON backers_and_creators ON backers.backer_id = backers_and_creators.backer_id
    
      

Always use CTEs over inlined subqueries.

  
  
  

----------

  

Reference:

-   [https://gist.github.com/fredbenenson/7bb92718e19138c20591#file-kickstarter_sql_style_guide-md](https://gist.github.com/fredbenenson/7bb92718e19138c20591#file-kickstarter_sql_style_guide-md)






