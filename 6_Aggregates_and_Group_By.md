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

By default aggregate functions use every input value. But most allow you to work on the unique values in the input. You do this with the keyword distinct.

For example, you can find the number of different values in the colour column by passing "distinct colour" to count. There are three colours (red, green, and blue). So doing this returns three:

```sql
select count ( distinct colour ) number_of_different_colours
from bricks;
```

You can explicitly tell the function to process every row with the keyword all. You can also use unique as a synonym for distinct:

```sql
select count ( all colour ) total_number_of_rows,
       count ( distinct colour ) number_of_different_colours,
       count ( unique colour ) number_of_unique_colours
from bricks;
```

You can also use distinct in most stats functions, like sum and avg. The brick table stores the weights 1, 1, 1, 2, 2, and 3. Which has the distinct values 1, 2, and 3. So the overall weight and mean are 10 and 1.66... respectively. But the distinct sum and weight are 6 and 2:

```sql
select sum ( weight ) total_weight, sum ( distinct weight ) sum_of_unique_weights,
       avg ( weight ) overall_mean, avg ( distinct weight ) mean_of_unique_weights
from bricks;
```

Not all aggregates support distinct. Refer to the documentation for full details.

### Try it
Complete the following query to return the:

- Number of different shapes
- The standard deviation (stddev) of the unique weights

```sql
select /* TODO */ number_of_shapes,
       /* TODO */ distinct_weight_stddev
from   bricks;
```

This is a solution:

```sql
select count ( distinct shape ) number_of_different_shape,
       stddev ( distinct weight ) stddev_of_the_unique_weight
from bricks;
```

## 4. Grouping Aggregates

As well as the overall total, you can split the results into separate groups. You do this with group by. This returns one row for each combination of values in the group by columns.

For example, the following returns the number of rows for each colour:

```sql
select colour, count(*)
from bricks
group by colour;
```
You don't need to include all the columns in the group by in your select clause. The following splits the rows by colours as above. But excludes colour from the output:

```sql
select count (*)
from bricks
group by colour;
```

This can be confusing, so it's a good idea to include all the grouping columns in your select.

But the reverse isn't true! __All unaggregated values in your select clause must be in the group by.__

So the following will raise an exception because shape is in the select, but not the group by:

```sql
select colour, shape, count (*)
from bricks
group by colour;
```

You can group by many columns. The following returns the number of rows for each shape and weight:

```sql
select shape, weight, count(*)
from bricks
group by shape, weight;
```

### Try it
Complete the following query to return the total weight for each shape stored in the bricks table:

```sql
select shape, /* TODO */ shape_weight
from   bricks
/* TODO */;
```

```sql
select shape, sum ( weight ) total_weight
from bricks
group by shape;
```

## 5. Filtering Aggregates

__The database processes group by after the where clause.__ So the following excludes rows with a weight <= 1 from the count:

```sql
select colour, count (*)
from bricks
where weight > 1
group by colour;
```

You can only filter unaggregated values in the where clause. If you include aggregate functions here, you'll get an error.

For example, the following tries to return the colours which have more than one row:

```sql
select coulor, count (*)
from bricks
where count (*) > 1
group by colour;
```

But it throws an ORA-00934 error.

to filter aggregate functions, use the having clause. This can go before or after group by. So the following both return the colours which have more than one row in bricks:

```sql
select colour, count (*)
from bricks
having count (*) > 1
group by colour;

select colour, count (*)
from bricks
group by colour
having count (*) > 1;
```

You can use different functions in your select and having clauses. For example, the following finds the colours which have a total weight greater than 1. And returns how many rows there are for each of these colours:

```sql
select colour, count (*)
from bricks
having sum ( weight ) > 1
group by colour;
```

### Try it
Complete the following query to find the shapes which have a total weight less than four:

```sql
select shape, sum ( weight )
from bricks
group by shape
having sum ( weight ) < 4;
```

## 6. Generating Subtotals

You can also use group by to generate subtotals for each value in the grouping columns. You can do this with rollup or cube.

### Rollup
Rollup generates the subtotals for the columns within it, working from right to left. So a rollup of:

rollup ( colour, shape )

calculates:
- Totals for each ( colour, shape ) pair
- Totals for each colour
- The grand total

The groups using every column in the rollup are regular rows. Those based on a subset are supperaggregate rows. For these superaggreagtes, the database returns null for the grouped columns. So the colour totals show null for shape. And the grand total null for colour and shape:

```sql
select colour, shape, count (*)
from bricks
group by rollup (colour, shape);
```

You can combine a rollup with non-rolledup columns. In this case the "grand total" is for the columns outside the rollup. The following calculates the total for each ( colour, shape ) pair and the number of rows for each colour:

```sql
select colour, shape, count (*)
from bricks
group by colour, rollup ( shape );
```

Rollup calculates N+1 groupings, where N is the number of expressions in the rollup. So a rollup with three columns returns 3+1 = 4 groupings.

### Cube

Cube calculates the subtotals for every combination of columns within it. So if you use:

cube ( colour, shape )

You get groupings for:

- Each ( colour, shape ) pair
- Each colour
- Each shape
- All the rows in the table

```sql
select colour, shape, count (*)
from bricks
group by cube (colour, shape);
```

Cube calculates 2^N groupings, where N is the number of expressions in the cube. So a cube with three columns returns 2^3 = 8 groupings.
