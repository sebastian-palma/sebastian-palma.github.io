+++
title = "The SQL Join clause"
weight = 0
date = 2019-09-20
insert_anchor_links = "right"
[taxonomies]
categories = ["database", "sql", "join"]
+++

<br />
<br />

The term _join_ concerning to databases appeared with the publishing of the paper "A Relational Model of Data for Large Shared Data Banks" by Edgar F. Codd (IBM Research Laboratory) in June of 1970. After years working and dealing with data banks using the Hierarchical and Network data models, and seeing the inflexibility that programs developed at that time presented for their maintainers.

In the paper, Codd enumerates several problems with the data and access path dependencies in the systems and introduces his idea of a relational view of data accompanied by the early definitions of the normal form to avoid redundancy and get consistency.

The term relation is the main point from his work and is what we use today when working with relational databases applying the relational model. The elimination of non-simple domains (structures holding a single set of data, e.g a column) by taking its primary key and expanding each of the subordinate relations by inserting the primary key is what he called normalization.

Considering the following example he presents the differences between an un-normalized set and a normalized set:

"_jobhistory_ and _children_ are non simple domains of the relation _employee_. _salaryhistory_ is a non simple domain of the relation _jobhistory_."

<br />

{% katex(block=true) %}
\text{employee}\\
\text{|}\\
\text{-----------------------------------------------------------}\\
\text{|}~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\text{|}\\
\text{jobhistory}~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\text{children}\\
\text{|}~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
\text{salaryhistory}~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{% end %}

<br />

Un-normalized set:

{% katex(block=true) %}
employee~(\textbf{man}\#,~name,~birthdate,~jobhistory,~children)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
jobhistory~(jobdate,~title,~salaryhistory)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
salaryhistory~(salarydate,~salary)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
children~(childname,~birthyear)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
{% end %}

Normalized set:

{% katex(block=true) %}
employee^{\prime}~(\textbf{man}\#,~name,~birthdate)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
jobhistory^{\prime}~(\textbf{man}\#,~jobdate,~title)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
salaryhistory^{\prime}~(\textbf{man}\#,~jobdate,~salarydate,~salary)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
children^{\prime}~(\textbf{man}\#,~childname,~birthyear)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
{% end %}

<br>

The normalization starts by taking the primary key of the topmost relation and insert it into the immediately subordinate relations. Now, the primary key of each expanded relation is the primary key before expansion augmented by the primary key copied down from the parent relation. After that, all nonsimple domains from the parent relation and the top node of the tree are removed. The subsequence of operations is then applied to each remaining sub-tree.

This eliminates duplicate data and adds simplicity not only for storage purposes, but also for the communication of bulk data between systems that use widely different representations of the data. 

The term _relation_ is presented as the core of the paper (with the term relation in its mathematical sense):

\\(  \\)


> Given sets \\( S_1,S_2,..., S_n \\) (which not necessarily must be distinct), \\( S \\) is a relation on these \\( n \\) sets if it's a set of \\( n-tuples \\) each of which has its first element from \\( S_1 \\), its second element from \\( S_2 \\), and so on.
> <br />
> <br />
> \\( R \\) is a subset of the Cartesian product \\( S_1 \\) X \\( S_2 \\) X \\( ... \\) X \\( S_n \\).
> <br />
> <br />
> Referring to \\( S_1 \\) as the \\( jth \\) domain of \\( R \\). \\( R \\) is said to have degree \\( n \\). Relations of degree 1 are often called \\( unary \\), degree 2 \\( binary \\), degree 3 \\( ternary \\) and degree \\( n \\) \\( n-ary \\).

Since relations are sets, all of the usual set operations apply to them. Codd explains 5 possible ones. Among them is the _join_ operation.

Suppose we're given two binary relations, with a domain in common. Under what circumstances can we combine these relations to form a ternary relation which preserves all of the information in the given relations?

There are two relations \\( R \\) and \\( S \\), which can be joined without loss of information.

| \\( R \\)  | (supplier | part) | \\( S \\)  | (part | project) |
| ---- | --------- | ----- | ---- | ----- | -------- |
|      | 1         | 1     |      | 1     | 1        |
|      | 2         | 1     |      | 1     | 2        |
|      | 2         | 2     |      | 2     | 1        |



A binary relation \\( R \\) is _joinable_ with a binary relation \\( S \\) if there exists a ternary relation \\( U \\) such that \\( a_{12}(U) = R \\) and \\( a_{23}(U) = S \\). Any one of these ternary relations is called a join of \\( R \\) with \\( S \\).

If \\( R \\), \\( S \\) are binary relations such that \\( a_2(R) = a_1(S) \\), then \\( R \\) is able to be joined with \\( S \\).

One join that always exists in such a case is the _natural join_ of \\( R \\) with \\( S \\) defined as:


{% katex(block=true) %}
R * S = { (a, b, c):R(a, b) \ \wedge S(b, c) }
{% end %}


Where \\( R(a, b) \\) has the value _true_ if \\( (a, b) \\) is a member of \\( R \\) and similarly for \\( S(b, c) \\). It's immediate that


{% katex(block=true) %}
a_{12}(R * S) = R
{% end %}
and

{% katex(block=true) %}
a_{23}(R * S) = S
{% end %}



##### What's a Join?

In simple words, a _join_ is a constituent component of a statement and/or a query used to collate the data (or rows) from one or more tables  based on a common field between them. 

It's the act of connecting related tables or sets of data based on key values (column, column value). For it to work in a relational model is needed that all tables contain a unique identifier (primary key), and this way any related table contains a copy (same value, same data type) of that unique identifier (foreign key).



##### Theta Join. Equijoin

To start explaining what the _theta join_ is, first is needed to know about _the product_.  The product is (as its name says) the product between two tables \\( R \\) and \\( S \\), expressed as \\( R \times S \\). Pretty much as the Cartesian product of both \\( R \\) and \\( S \\) is obtained.

\\( R \times S \\) forms a table by concatenating all rows from the table \\( R \\) with all rows from the table \\( S \\). The columns are all the columns of both \\( R \\) and \\( S \\) tables correspondingly, and the total of rows is the sum of the total rows in \\( R \\) and the total rows in \\( S \\).

The most common way to get a product table is by using a _nested loop_ algorithm. To do so the first row of \\( R \\) it's merged with the first row of \\( S \\), then with the second row of \\( S \\). This way subsequently until reaching the last element in \\( S \\).  The operation is repeated with the second row of \\( R \\), the third, fourth, etc.

The total rows of _the product_ of \\( R \times S \\) are \\( x \times y \\) rows, where \\( x \\) is the total of rows in \\( R \\) and \\( y \\) is the total of rows in \\( S \\). As an example, the total of rows in users is 6 and the total of rows in posts is 15, _the product_ contains 90 rows.

The result of executing a _select_ operation on _the product_ is represented as the _theta join_. The symbol used to represent it is \\( |x|_0 \\).

For the tables \\( R \\) and \\( S \\), the theta join is defined as:


$$
R~|x|_0~S~=~\sigma_0(R~\times~S)
$$


For instance, we can get the rows from the product of \\( R \\) and \\( S \\) where _words_count_ is greater than 2771 (second post). This is represented as:


$$
\sigma_{words\_count~>~2771}~(users~\times~posts)
$$


In PostgreSQL this might be as:

```sql
SELECT *
FROM users
CROSS JOIN posts
WHERE words_count > 2771;
```

The _cross join_ clause used allows us to produce the Cartesian product between two or more tables.

A _theta join_ is then the result of a selection operation in _the product_ by using any binary relational operator ($> \\), \\( \geqslant \\), \\( = \\), \\( \neq \\), \\( \leqslant \\), \\( <$)

If the operator used is equality (\\( = \\)), then the join is also called _equijoin_.



##### Natural Join

The natural join is the easiest part to start explaining the concept of SQL join. Given the query "Get all user names and post titles from users who have written a post" the following SQL returns what we need:

```sql
SELECT name, title
FROM users, posts
WHERE users.user_id = posts.user_id;
```

What we're doing here is trying to match a certain condition to pick up the rows from both tables when it succeeds. The condition required is _users.user_id_ must be equal to _posts.user_id_. These two columns are the only ones in both tables with the same name. And, for the sake of simplicity SQL supports an operation called _natural join_ (and several different ways to join information from 1 or more tables).

The last query was nothing more than a Cartesian product with a where clause where we extracted information from two tables, without really creating a relationship between them. What we need now is to get the job done but using a join clause.

The natural join operation receives two (or more) tables and returns a third table as its result, often referred to as a "logical table". It considers only those pairs of rows with the same value on those columns in both tables, as opposed to the Cartesian product between two tables, which links each row of the table to the left of the operator with every row of the table to the right of the operator.

The example shown before can be expressed using the natural join operation as:



```sql
SELECT name, title
FROM users NATURAL JOIN posts;
```



| name               | title                                                        |
| ------------------ | ------------------------------------------------------------ |
| Diego Barton       | OpenAI Hide-and-Seek Findings, the Systems Perspective       |
| Salina Hill       | Lilly Singhs Late Night Show Is an Uneasy Compromise for Everyone |
| Salina Hill       | Can You Trust Facebook with your Love Life?                  |
| Diego Barton       | How Netflix Binges and Google Translate Helped Me Find Love  |
| Earlie Harber      | The Mystical Side of A.I.                                    |
| Salina Hill       | Start Before You're Ready                                     |
| Landon Legros      | Why Reading Poetry Can Make You a Better Leader              |
| Douglas Bartoletti | Stop Copying Your Heroes                                     |
| Salina Hill       | The Case for Being a Multi-Hyphenate                         |
| Landon Legros      | To Do Better Work, Change Your Environment                   |
| Salina Hill       | My Startup Could've Exploited Gig Economy Workers - Heres Why We Didn't |
| Landon Legros      | The Women Behind Controversial At-Home Rape Kits Speak Out   |
| Douglas Bartoletti | Why Your Startup Isn't Being Funded and What to Do About It   |
| Diego Barton       | Daredevil Unicorns: Why Juul, Uber, And Other Companies Play With Fire |
| Salina Hill       | The Surprisingly Effective Impact of Becoming a Connector Manager |



It results in the same, but it considers only those pairs of rows with both users and posts having the same values for the column in common - _user_id_. Giving a table with only 15 rows where there are no repeated values, nor for users, nor posts.

The order in which the columns are returned when it isn't specified follows a particular rule; first are the columns in common with the tables used, second the columns existing only to the left of the _natural join_ sentence and lastly the columns existing only in the table to the right of the operator. The result of a _natural join_ is a new relation.

The query is also executed in a specific order, first by its _from_ clause, following by the _where_ clause and then _select_ one.

```sql
SELECT *
FROM users
NATURAL JOIN posts LIMIT 5;
```

| user_id | name      | post_id | title                 | words_count | publishing_year | rating | category_id |
| ------- | --------- | ------- | --------------------- | ----------- | --------------- | ------ | ----------- |
| 5       | Diego B.  | 1       | OpenAI Hide-and...    | 1884        | 2017            | 3      | 3           |
| 3       | Salina H. | 2       | Lilly Singhs Late...  | 2771        | 2016            | 5      | 2           |
| 3       | Salina H. | 3       | Can You Trust...      | 3519        | 2019            | 4      | 3           |
| 5       | Diego B.  | 4       | How Netflix Binges... | 2643        | 2018            | 4      | 2           |
| 2       | Earlie H. | 5       | The Mystical Side...  | 2819        | 2016            | 2      | 1           |

The `from` clause can easily be chained for multiple table names, following the form:

{% katex(block=true) %}
\text{SELECT}~A_1,~A_2,~...,~A_n~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
\text{FROM}~r_1~\text{NATURAL JOIN}~r_2~\text{NATURAL JOIN}~...~\text{NATURAL JOIN}~r_m~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
\text{WHERE}~P;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\
{% end %}


Consider the case where we want to add the category of those posts written by each user. It'd involve adding a third table to the operation. This can be done as seen before, appending the needed table name to the _from_ clause. A _natural join_ will first compute the table to the left of the operator plus the first table to the right of the operator, resulting in a Cartesian product used to compute the _join_ with the following table to the right of the _natural join_.

````sql
SELECT users.name, posts.title, categories.name
FROM users
NATURAL JOIN posts, categories
WHERE posts.category_id = categories.category_id LIMIT 5;
````



| name          | title                          | name       |
| ------------- | ------------------------------ | ---------- |
| Diego Barton  | OpenAI Hide-and-Seek Findings, | startups   |
| Salina Hill  | Lilly Singhs Late Night Show I | creativity |
| Salina Hill  | Can You Trust Facebook with yo | startups   |
| Diego Barton  | How Netflix Binges and Google  | creativity |
| Earlie Harber | The Mystical Side of A.I.      | technology |

The _natural join_ result is affected by the extraction of those rows on the left side of the operator matching by its identifier with those rows to the right of the operator. In the example used, the _where_ clause  tries to satisfy the match condition using the common columns between _posts_ and _categories_, but the _posts.category_id_  points to the _natural join_ result of _users_ and _posts_.

The syntax shown before is not a short version of:

{% katex(block=true) %}
\text{NATURAL JOIN}~r_2~\text{NATURAL JOIN}~...~\text{NATURAL JOIN}~r_m \\
{% end %}

The result of the following query using the referred syntax returns no rows:

```sql
SELECT users.name, posts.title, categories.name
FROM users
NATURAL JOIN posts
NATURAL JOIN categories;
```

The reason is that the result of the _natural join_ between _users_ and _posts_ contains the columns are _user_id_, _name_, _post_id_, _title_, _words_count_, _publishing_year_, _rating_, and _category_id_. While the columns present in a _natural join_ between _posts_ and _categories_ are _name_, _user_id_, _category_id_. A _natural join_ would evidently ask for the column _category_id_ to be present in the _users_ table, which isn't. So the query shown above omits all those results where the user doesn't have a category_id, resulting in 0 rows.

To get the expected result from the last query we can use the _join ... using_ operation. Which expects one or more columns to be added to the _using_ clause. Then the tables being joined must contain those columns:

```sql
SELECT users.name, posts.title, categories.name
FROM (users NATURAL JOIN posts)
JOIN categories USING (category_id);
```

The operation:

$$
r_1~\text{join}~\text{using}(A_1,~A_2)
$$

Is similar to the _natural join_:

$$
r_1~\text{natural join}~r_2
$$

Except that a pair of rows \\( s_1 \\) from \\( r_1 \\) and \\( s_2 \\) from \\( r_2 \\) match if \\( s_2.A_1 = s_2.A_1 \\) and \\( s_1.A_2 = s_2.A_2 \\). If \\( r_1 \\) and \\( r_2 \\) both have a column \\( A_3 \\). It's not mandatory that \\( s_1.A_3 = s_2.A_3 \\).



##### Inner Join

There are different ways to perform a _join_ as per the SQL Standard. The most used and common is the _inner join_.

An _inner join_ linking the tables _users_ and _posts_ (using the _user_id_ column from both tables) returns only those rows from the table _users_ "linked" with those from the _posts_ table, meaning a link as the match between the value of _user_id_ in both tables. This way all rows that aren't able to be linked aren't added to the "logical table".

The main part in an _inner join_ is the one that tells the database how to perform the _join_; the _on_ or _using_ clauses right after the second table in the condition.

The operation is executed first by logically combining every row from the table to the left side of the operator with every row of the table to the right side of the operator - the Cartesian product between both tables. Then it applies the criteria used in the _on_ or _using_ clauses to select those rows that match and return them. The search (or match condition) in  the _on_ clause tells to the _join_ the logical test that must be true to return any two linked rows.

Let's use the example used in the _natural join_ section:  "Get all users names and post titles from users who have written a post".

```sql
SELECT name, title
FROM users
INNER JOIN posts
ON users.user_id = posts.user_id;
```

When using multiple tables in a _from_ clause is preferred to always do explicit mention of the tables holding the columns we're using. The column _name_ is only present in the _users_ table, as well as the _title_ column only in the _posts_ table, that's why in this query isn't needed to prefix the table names before the column names. But we will always do this with the _on_ clause.

We can achieve the same result with the _using_ clause. Where we don't have to add the table names as the columns prefixes, as it takes the matching pair of the column from both tables:

```sql
SELECT name, title
FROM users
INNER JOIN posts
USING(user_id);
```

In both cases, the database evaluates the complete _join_ clause before to start retrieving rows. In most of the cases, a row \\( a \\)  is first fetched from the table to the left side of the operator. Then the database makes use of an internal link (an index if defined) to fastly find any rows in the table to the right side of the operator that matches the row \\( a \\) before moving to the next row in the current table.



##### Join Conditions

Besides the _natural join_, SQL supports a similar kind of join where you can make use of arbitrary conditions.

An _on_ condition allows the tables to be joined over a general predicate. The predicate is written like using a _where_ clause predicate, the only difference is the use of the keyword _on_, instead of _where_. Similar to the _using_ condition, _on_ is always added at the end of a _join_ expression.

To get all the columns from both users and posts tables where the _users.user_id_ column is equal to the _posts.user_id_ (with a limit of 5 for the sake of brevity) we would use a query like the following:

```sql
SELECT *
FROM users JOIN posts
ON users.user_id = posts.user_id LIMIT 5;
```

(the table name prefixes are used to disambiguate the columns used for the match in both tables)

| user_id | name      | post_id | title                      | words_count | publishing_year | rating | category_id | user_id |
| ------- | --------- | ------- | -------------------------- | ----------- | --------------- | ------ | ----------- | ------- |
| 5       | Diego B.  | 1       | OpenAI Hide-and-Seek ...   | 1884        | 2017            | 3      | 3           | 5       |
| 3       | Salina H. | 2       | Lilly Singhs Late ...      | 2771        | 2016            | 5      | 2           | 3       |
| 3       | Salina H. | 3       | Can You Trust Facebook ... | 3519        | 2019            | 4      | 3           | 3       |
| 5       | Diego B.  | 4       | How Netflix Binges ...     | 2643        | 2018            | 4      | 2           | 5       |
| 2       | Earlie H. | 5       | The Mystical Side ...      | 2819        | 2016            | 2      | 1           | 2       |

The _on_ condition in the above query establishes that a row from _users_ matches a row from _posts_ if the value of _user_id_ is equal in both rows. There's an evident similarity of the _on_ clause used and the _natural join_ expressions seen before. With the _natural join_ rows in the _users_ table had to match with the ones in the _posts_ table. The result of a _join_ adds twice the column from the condition (first and last columns), while the _natural join_ result doesn't.

An alternative to the last query would be using _where_ to check if both columns in each row of both tables match:

```
SELECT *
FROM users, posts
WHERE users.user_id = posts.user_id;
```

In this case, the _user_id_ is also twice in the result. To avoid getting unnecessary columns, listing the needed columns is always an option.

Using the _on_ condition allows us to express any SQL predicate, thus using _join_  expressions with _on_ conditions we get more flexibility while expressing the matching conditions we need.

The _on_ condition might seem useless as we can get a similar join result using _where_ instead of a _join ... on_ operation. But there are two good reasons for the _on_ condition to exist. In an _outer join_ operation, the _where_ conditions don't work as seen before. The _on_ clause also offers more readability at the moment of composing queries since is easier to read and understand a query when the _on_ clause holds the _join_ condition, and the rest does it the _where_ clause.

It's very common to _join_ tables by using the primary key from one table in conjunction with its foreign key in another table. But this isn't mandatory. A _join_ can be performed as long as both columns are of the same data type. Any column of type char can be joined with another column of the same type, as well as integer columns from table \\( A \\) to an integer column in a table \\( B \\), and so on.

The latter is perfectly valid but is always up to the user to make them work in a valid context and under the same meaning.

A recommendation when working with _join_, no matter the number of tables and _on_ conditions being used, it's always better to explicitly state the type of _join_ to use and to qualify the used column names with the name of their parents' tables.



##### Outer Joins

If we need to get all the users and display all their columns plus the columns from the posts table in case they have written a post, a query like the following should work:

```sql
SELECT *
FROM users NATURAL JOIN posts;
```

But it doesn't. If there are users that haven't written a post, then the row in the _users_ table corresponding to that user won't match the condition of the _natural join_, which implies the _user_id_ in the row in the table to the left of the operator must match with the _user_id_  of the row of the table to the right of the operator (in this case by the _user_id_), thus that row won't be taken into account for the result given.

The _outer join_ operation works in a similar way to the _join_ operations, but instead of discarding the rows not matching the specified criteria, it preserves them. It does so by creating rows with _NULL_ values in the result of the operation.

To exemplify, in the latter example there's one row from the _users_ table that doesn't have any post. That's to say, there's no row in the _posts_ table with _user_id_ equal to the id of the only user that hasn't written a post. So, when executing the query containing the _outer join_, a row containing all the values from the user that didn't write a post is added to the join result. In the same way, all the values corresponding to the columns in the _posts_ table are added to the row created before, but this time with all its values as _NULL_. This way the row for that user is preserved in the result of the _outer join_.

##### Outer Join forms

- A _left outer join_ preserves only the rows from the table to the left of the operation (_left outer join_).

- A _right outer join_ preserves only the rows from the table to the right of the operation (_right outer join_).

- A _full outer join_ preserves the rows in both tables.

<br />

The join operations that don't preserve the rows that don't match are called _inner-join_ operations. Their exact opposite is called _outer-join_ operations.

An _outer join_ first computes its _inner join_ operations in sequential order, then for every row \\( a \\) in the table to the left of the operator that doesn't match any row in the table to the right of the operator in the _inner join_, it adds a row _b_ to the result of the _join_ following the next rules:

- All the columns of the _b_ row from the table to the left of the operator have the values of the row _a_.
- All the remaining columns of _b_ are then _NULL_ values (this depends on the RDBMS being used, PostgreSQL returns an empty column).

The difference between a _natural ... outer join_ and a _... outer join ... on_ is as shown before, they return the same columns and rows, but the last one adds twice the column used for the match condition:

```sql
SELECT *                                   SELECT *
FROM users                                 FROM users
NATURAL LEFT OUTER JOIN posts;             LEFT OUTER JOIN posts
                                           ON users.user_id = posts.user_id;
```

Both _left outer join_ and _right outer join_ are uniform operations. Rows in the right-hand-side table that don't match any other row in the left-hand-side table take _NULL_ values and are appended to the result of the _right outer join_. 

The result of a _left outer join_ operation can be achieved from a _right outer join_ if we interchange the order of the tables in the operation:

```sql
SELECT *                                   SELECT *                     
FROM users                                 FROM posts 
NATURAL LEFT OUTER JOIN posts;             NATURAL RIGHT OUTER JOIN users;
```

What differs is just the order in which the columns are listed.

##### Full Outer Join

The combination of a _left_ and _right_ _outer-join_ types produces a _full outer join_. It works similarly to an _outer join_, but right after the operation computes the result of the inner join, it adds _NULL_ values to those rows from the table to the left side of the operator that doesn't match with any row from the table to the right side of the operator, and pushes them to the result of the operation. It does the same with the rows of the table to the right side of the operator that doesn't match with any row from the table to the left side of the operator. In the same way, it adds those rows to the result.

In terms of correspondence, both the _left outer join_ and the _right outer join_ can be applied as their _union_.

As with _left_ and _right_ _outer_ joins, the _on_ clause can be used to control the columns we're expecting to match. The _on_ and _where_ clauses have different behavior for _outer join_. This is because _outer join_ adds rows allowing _NULL_ values for those rows that don't match with the criteria specified in an _inner join_. The _join_ specification incorporates the _on_ condition on its own, but not the _where_ clause.



##### Joins and Conditions

_inner join_ is the name used to distinguish "normal" _joins_ from _outer joins_. The _inner_ keyword in an _inner join_ is optional, and thus _inner join_ is just used to emphasize the use of a normal join.

Both following queries are equivalent:

````sql
SELECT *                                   SELECT *
FROM users                                 FROM users
JOIN posts USING (user_id);                INNER JOIN posts USING (user_id);
````

The same equivalence is for _natural join_ and _natural inner join_.

Any _join_ form can be combined with any condition form, thinking on it as the Cartesian product between _join_ types and _join_ conditions.



##### Self-Join

A _self-join_  is joining a table with itself. It's used when a table contains a reference to the same table through the combination of a primary key and a foreign key. Both columns are held within the table being used supported by the referential integrity constraint.

Let's exemplify by adding a new column to _posts_ which makes mention to the same table to show the post that references the current post:

```sql
ALTER TABLE posts ADD COLUMN reference_post_id INTEGER REFERENCES posts(post_id);
```

We can assign any random post referrer to every post by using the referrer post_id value.

This way we can get all the posts that have been referenced by using an _inner join_ and selecting the _title_ from both tables and distinguishing them by using aliases:

```sql
SELECT a.title, b.title 
FROM posts a
INNER JOIN posts b ON a.post_id = b.reference_post_id;
```

Any possible condition used in a regular _join_ can also be used in a self-join.

{% references() %}
&nbsp;
{% end %}

[http://infolab.stanford.edu/~ullman/fcdb/aut07/slides/ra.pdf](http://infolab.stanford.edu/~ullman/fcdb/aut07/slides/ra.pdf)
<br /><br />
[https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf](https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf)
<br /><br />
[https://stackoverflow.com/questions/7870155/difference-between-a-theta-join-equijoin-and-natural-join](https://stackoverflow.com/questions/7870155/difference-between-a-theta-join-equijoin-and-natural-join)
<br /><br />
[https://www.pearson.com/us/higher-education/program/Viescas-SQL-Queries-for-Mere-Mortals-A-Hands-On-Guide-to-Data-Manipulation-in-SQL-4th-Edition/PGM1937355.html](https://www.pearson.com/us/higher-education/program/Viescas-SQL-Queries-for-Mere-Mortals-A-Hands-On-Guide-to-Data-Manipulation-in-SQL-4th-Edition/PGM1937355.html)
<br /><br />
[https://www.postgresql.org/docs/9.2/queries-table-expressions.html](https://www.postgresql.org/docs/9.2/queries-table-expressions.html)
<br /><br />
[https://www.db-book.com/db7/](https://www.db-book.com/db7/)
<br /><br />
[https://zeepedia.com/toc.php?database_management_systems&b=7](https://zeepedia.com/toc.php?database_management_systems&b=7)
<br /><br />
[https://en.wikipedia.org/wiki/Referential_integrity](https://en.wikipedia.org/wiki/Referential_integrity)
<br /><br />
[https://en.wikipedia.org/wiki/Relational_algebra](https://en.wikipedia.org/wiki/Relational_algebra)

{% end_references() %}
&nbsp;
{% end %}