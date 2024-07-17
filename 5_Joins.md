# Joining Tables

### Content

1. Introduction
2. Join Syntax: Oracle vs. ANSI
3. Cross Joins
4. Inner Joins
5. Outer Joins
6. Filtering Joins
7. Full Outer Joins

-----------------------------------------------------------------------------------------------------------------------

### Prerequisite SQL
```sql
create table toys (
    toy_id     integer,
    toy_name   varchar2(20),
    toy_colour varchar2(10)
);

create table bricks (
    brick_id     integer,
    brick_colour varchar2(10),
    brick_shape  varchar2(10)
);

insert into toys values (1, 'Miss Snuggles', 'pink');
insert into toys values (2, 'Cateasaurus', 'blue');
insert into toys values (3, 'Baby Turtle', 'green');

insert into bricks values (2, 'blue', 'cube');
insert into bricks values (3, 'green', 'cube');
insert into bricks values (4, 'blue', 'pyramid');

commit
```

## 1. Introduction

You can combine rows from two tables with a join. This tutorial explains the join methods using these two tables:

```sql
select * from toys;

select * from bricks;
```

## 2. Join Syntax: Oracle vs. ANSI

Oracle Database has two syntaxes for joining tables. The proprietary Oracle method. And the ANSI standard way.

Oracle syntax joins tables in the where clause. ANSI style has a separate join clause. This tutorial will show both methods.

We recommend you use ANSI syntax. This clearly separates the join and filter clauses. This can make your query easier to read, particularly with outer joins.

## 3. Cross Joins : Cartesian product

A cross join returns every row from the first table matched to every row in the second. This will always return the Cartesian product the two table's rows. I.e. the number of rows in the first table times the number in the second.

Toys and bricks both store three rows. So cross joining them returns 3 * 3 = 9 rows.

To cross join tables using Oracle syntax, simply list the tables in the from clause:

```sql
select *
from toys, bricks;
```

Using ANSI style, type cross join between the tables you want to combine:

```sql
select *
from toys
cross join bricks;
```

## 4. Inner Joins : Cartesian product + the condition

An inner join (or just join) links two tables. It compares values in one or more columns from each. It only returns rows which match the join conditions in both tables.

The simplest join checks if the values in a column from one table equal the values in a column from the other.

For example, you can join toys and bricks where the toy_id equals the brick_id. Only the ids 2 & 3 are in both tables. And there is only one row for each value in each table. So this join returns two rows:

```sql
select *
from toys, bricks
where toy_id = brick_id;
```
Using the ANSI way, the join criteria go in the on clause.


```sql
select * 
from  toys
inner join bricks -- or simply join bricks
on    toy_id = brick_id;
```

If you join on a column that has repeated values, you'll get many copies of the matching rows in the joined table. For example, there are two blue bricks. The toy Cuteasaurus is blue. So joining the tables on colour returns two copies of the row for Cuteasaurus:

```sql
select *
from toys
join bricks
on toy_colour = brick_colour;
```

You can also join on inequalities. For example, to return all the rows that have different colours, write:

```sql
select *
from toys
join bricks
on   toy_colour <> brick_colour;
```

Note there are no pink bricks. So this returns a copy of the pink toy (Miss Snuggles) for each row in bricks. So Miss Snuggles appears three times.

### Try it

Complete the query below to:

- Inner join toys and bricks
- Where the toy_id is greater than the brick_id

```sql
select * 
from   toys
join   bricks
/* TODO */
```

This is a solution:

```sql
select *
from toys
join bricks
on toy_id > brick_id;
```

![image](https://github.com/user-attachments/assets/a80751a4-fae9-45bb-a46b-fbd924590eff)


## 5. Outer Joins : entire on side (left or ... ) + all matching row (based on the condition) of other side



## 6. Filtering Joins
## 7. Full Outer Joins

```sql
```
