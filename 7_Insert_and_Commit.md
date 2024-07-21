# Insert and Commit : Adding and Saving Rows

### Content

1. Introduction
2. Single Row Insert
3. Multi-row Insert
4. Performance: Single Row vs. Multi-row
5. Saving DML Changes
6. Undoing DML
7. Savepoints
8. Multi-table Insert
9. Conditional Multi-table Insert

-----------------------------------------------------------------------------------------------------------------------

### Prerequisite SQL

```sql
create table toys (
    toy_id   integer,
    toy_name varchar2(100),
    colour   varchar2(10)
);

create table bricks (
    brick_id integer,
    colour   varchar2(10),
    shape    varchar2(10)
);
```

## 1. Introduction
Insert is a form of DML (Data Manipulation Language) statement. You use it to store values in a table.

This tutorial shows you how insert works using these two tables:

```sql
select * from toys;

select * from bricks;

select table_name, column_name, data_type
from user_tab_columns
where table_name in ('TOYS', 'BRICKS')
order by table_name, column_id
```

## 2. Single Row Insert

You can add one row at a time to a table with the insert clause. This takes the form:

insert into <table_name> ( col1, col2, ... )
  values ( 'value1', 'value2', ... )

The column list is optional. If you omit this, you must provide a value for every visible column in the table.

For example, the following adds a row for a toy named Miss Snuggles coloured pink with the toy_id 1 into the toy table:

```sql
insert into toys values (1, 'Miss Snuggles', 'pink');

select * from toys;
```

If the number of values doesn't match the number and data types of the target columns, you'll get an error. The insert below only provides two values. But the table has three columns. So it throws an exception:

```sql
insert into toys values (2, 'Baby Tutls');
```

This makes inserts without a column list brittle. If you change the columns in the table, the insert is likely to fail. It's much better to list the columns you're providing values for. For example:

```sql
insert into toys (toy_id, toy_name) values (2, 'Baby Tutle');

select * from toys;
```

This will still work if someone adds another column or removes the colour column from toys. Any table columns you don't provide a value for will store null.

### Try it!

Complete the following query to insert a row into the toy table with the toy_id 3 and colour red:

```sql

```

This is a solution:

```sql

```

## 3. Multi-row Insert



## 4. Performance: Single Row vs. Multi-row
## 5. Saving DML Changes
## 6. Undoing DML
## 7. Savepoints
## 8. Multi-table Insert
## 9. Conditional Multi-table Insert
