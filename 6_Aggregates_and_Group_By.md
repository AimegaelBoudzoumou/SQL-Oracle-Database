# Aggregating Rows

### Content

1. Introduction
2. Aggregate Functions
3. Distinct vs. All
4. Grouping Aggregates
5. Filtering Aggregates
6. Generating Subtotals

-----------------------------------------------------------------------------------------------------------------------

### Prerequisite SQL

```sql
create table bricks (
    colour varchar2(10),
    sape   varchar2(10),
    weight integer
);

insert into bricks values ('red', 'cube', 1);
insert into bricks values ('red', 'pyramid', 2);
insert into bricks values ('red', 'cuboid', 1);

insert into bricks values ('blue', 'cube', 1);
insert into bricks values ('blue', 'pyramid', 2);

insert into bricks values ('green', 'cube', 3);

commit;
```

## 1. Introduction

This tutorial shows you how to summarize your data using aggregate functions and group by. It uses the following table as its source:

```sql
select * from bricks;
```

## 2. Aggregate Functions

Aggregate functions combine many rows into one. The query returns one row for each group. If you use these without group by, you have one group. So the query will return one row.

For example, count() returns the number of rows the query processed. So this query gets one row, showing you how many rows there are in the bricks table:

```sql
select count(*)
from bricks;
```

Count is unusual for accepting *. This means return the total number of rows. You can also pass an expression (column) to it. This returns the number of non-null rows for the expression. For example, the following shows you how many rows where colour is not null:

```sql
select count ( colour ) from bricks;
```

All other (non-count) aggregates use an expression. Oracle Database includes many statistical aggregate functions. Common ones include:

- Sum: the result of adding up all the values for the expression in the group
- Min: the smallest value in the group
- Max: the largest value in the group
- Avg: the arithmetic mean of all the values in the group
- Stddev: the standard deviation
- Median: the middle value in the data set
- Variance: the statistical variance of the values
- Stats_mode: the most common value in the group

The following query shows these in action:

```sql
select sum ( weight ), min ( weight ), max ( weight ),
       avg ( weight ), stddev ( weight ),
       median ( weight ), variance ( weight),
       stats_mode ( weight )
from bricks;
```

There are also many other, more specialized aggregates. You can find a [complete list of aggregate functions](https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/Aggregate-Functions.html#GUID-62BE676B-AF18-4E63-BD14-25206FEA0848) in the docs.

## 3. Distinct vs. All



## 4. Grouping Aggregates
## 5. Filtering Aggregates
## 6. Generating Subtotals

```sql
```
