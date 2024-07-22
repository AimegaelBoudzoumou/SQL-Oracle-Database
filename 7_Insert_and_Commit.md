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
insert into toys /* TODO */ values ( 3, 'red' );
```

This is a solution:

```sql
insert into toys (toy_id, colour) values (3, 'red');

select * from toys
where toy_id = 3;
```

## 3. Multi-row Insert

You can add many rows in one statement. To do this, you insert the output of a query.

For example, the insert below gets all the toy_ids in the toys table. Then inserts them into the bricks table:

```sql
select * from toys;

insert into bricks ( brick_id )
	select toy_id
	from toys;

select * from bricks;
```

Like single-row inserts, the number of columns in the query and target table must match. The data types of the columns in each position must also be compatible.

The toys table has three columns. But the query only returns one column. So the following statement will fail (**ORA-00913: too many values**):

```sql
insert into bricks ( bircks_id )
	select *
	from toys;
```

So, as with standalone queries, it's a good idea to list the columns in your select clause.

## 4. Performance: Single Row vs. Multi-row
In general it's better to combine SQL commands into as few statements as possible. This will give the best performance.

For example, the following compares the performance of single-row vs multi-row inserts. Both add 50,000 rows. Single-row adds one row 50,000 times. Multi-row adds 50,000 rows in one go. The multi-row version will be 10x to 100x faster!

```sql
declare 
	start_time   pls_integer;
	insert_count pls_integer := 5000;
begin
	start_time := dbms_utility.get_time ();

	for i in 1 .. insert_count loop
		insert into bricks
		values (i, 'red', 'cube');
	end loop;

	dbms_output.put_line ( 'Single-row duration = ' || (dbms_utility.get_time() - start_time) );

	rollback;

	start_time := dbms_utility.get_time();

	insert into bricks
		select level, 'red', 'cube' from dual
		connect by level <= insert_count;

	dbms_output.put_line ( 'Single-row duration = ' || ( dbms_utility.get_time() - start_time) );

	rollback;
end;
/
```

So use multi-row inserts where you can!

### Try it

Run these inserts:

```sql
insert into toys (toy_id, colour) values (4,'blue');
insert into toys (toy_id, colour) values (5, 'green');
```

Then complete this insert statement to add the rows for toy_ids 4 & 5 to bricks. It should store toy_id values in brick_id. And toy colours in brick colours.

```sl
insert /* TODO */
  select toy_id, colour
  from   toys
  where  toy_id in ( 4, 5 );
```

This is a solution:

```slq
insert into bricks (brick_id, colour)
	select toy_id, colour
	from toys
	where toy_id IN (4, 5);

select * from bricks
where brick_id in (4, 5);
```

## 5. Saving DML Changes
## 6. Undoing DML
## 7. Savepoints
## 8. Multi-table Insert
## 9. Conditional Multi-table Insert
