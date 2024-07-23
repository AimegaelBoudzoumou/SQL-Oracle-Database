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

```sql
insert /* TODO */
  select toy_id, colour
  from   toys
  where  toy_id in ( 4, 5 );
```

This is a solution:

```sql
insert into bricks (brick_id, colour)
	select toy_id, colour
	from toys
	where toy_id IN (4, 5);

select * from bricks
where brick_id in (4, 5);
```

## 5. Saving DML Changes
To save changes to the database, you must issue a commit. So to ensure you preserve the row you inserted, commit afterwards:

```sql
insert into toys (toy_id, toy_name, colour)
  values (6, 'Green Rabbit', 'green');

commit;

select *
from toys
where toy_id = 6;
```

The row is only visible to other users after you commit. Before this point, only your session (database connection) can see the new data.

Many tools have an autocommit property. This will do the commit for you. Either after each call to the database. Or when the session ends.

LiveSQL commits after each click of the Run button completes.

Ensure you check how this is set. If autocommit is on and you think it isn't (or vice-versa) it can lead to saving unwanted changes!

How to do this is specific to your tools and programming language. Refer to their documentation to find out how to set the autocommit property.

## 6. Undoing DML

Between inserting a row and committing it, your code may throw an exception. So you may wish to undo the change. You can do this with rollback. Rollback reverts all the changes since your last commit.

Say you added a row for Pink Rabbit. But realize you made a mistake, so want to remove it. The following does that:

```sql
insert into toys (toy_id, toy_name, colour)
  values (7, 'Pink Rabbit', 'pink');

select * from toys
where toy_id = 7;

rollback;

select * from toys
where toy_id = 7;
```

When you issue a rollback, the database will undo all changes you made since your last commit. If you commit between insert and rollback, rollback does nothing. And you need to run a delete to remove the row.

Note: Please ensure the SQL Worksheet includes all insert and rollback statements when you click Run. Each time you press run, Live SQL runs an implicit commit at the end of your statements so if you run each statement one at a time, you will not be able to rollback.

For example, the following commits the change before the rollback - so the row for Pink Rabbit remains in the table:

```sql
insert into toys (toy_id, toy_name, colour)
  values (7, 'Pink Rabbit', 'pink');

select * from toys
where toy_id = 7;

commit;
rollback;

select * from toys
where toy_id = 7;
```

Clicking one button in your application may fire many DML statements. You should defer committing until after processing them all. This allows you to rollback the changes if an error happens part way through. This ensures all other users of the application either see all the changes or none of them.

## 7. Savepoints
Your code will often add rows to many tables in one action. If there is an error part-way through, you may want to undo some - but not all - the changes since the last commit.

To help with this you can create savepoints. These are checkpoints. You can undo all changes made after it. And preserve those made beforehand.

To do this, first create the checkpoint with the savepoint command. Give it a name to refer to later:

```sql
exec savepoint save_this;
```

Note: exec is a requirement for LiveSQL. You do not need this prefix in other environments.

To undo the changes after the savepoint, add the clause "to savepoint" after rollback. Give the name of the savepoint you want to revert to. This undo will the changes you made after this:

```sql
exec savepoint save_this;
rollback to savepoint save_this;
```

Note that savepoints do NOT commit! If you issue an unqualified rollback, you'll still reverse all changes since the last commit. Even those made before the savepoint.

For example, the code below:

- Adds a row for toy_id 8
- Creates the savepoint after_six
- Then inserts toy_id 9

The rollback to savepoint only removes the row for toy_id 9. The final rollback at the end also removes toy_id 8:

```sql
insert into toys (toy_id, toy_name, colour)
  values ( 8, 'Pink Rabbit', 'pink' );

exec savepoint after_six;

insert into toys (toy_id, toy_name, colour)
  values ( 9, 'Purple Ninja', 'purple' );

select * from toys
where toy_id in ( 8, 9 );

rollback to savepoint after_six;

select * from toys
where toy_id in ( 8, 9 );

rollback;

select * from toys
where toy_id in ( 8, 9 );
```

You can create many savepoints in one call. And rollback to any of them. If you rollback to a savepoint this destroys any made after the one you name. But you can still go back to the earlier points.

For example, the following creates three savepoints. But the first rollback undoes the changes after second_sp. So third_sp is lost and you can no longer go back to it:

```sql
exec savepoint first_sp;
exec savepoint second_sp;
exec savepoint third_sp;

rollback to savepoint second_sp;
rollback to savepoint third_sp; /* Fails - third_sp no longer exists */
rollback to savepoint first_sp;
```

### Try it
Complete the following code, placing commits, savepoints, and rollbacks in the /* TODO */ sections, so that you:

- Commit the insert of toy_id 8
- Add rows for toy_id 9 & 10, but then remove the row for 10
- Then undo the insert for toy_id 9

```sql
insert into toys ( toy_id, toy_name, colour )
  values ( 8, 'Red Rabbit', 'red' );

/* TODO */
commit;

insert into toys (toy_id, toy_name, colour)
  values ( 9, 'Purple Ninja', 'red' );

/* TODO */
exec savepoint to_nine;

insert into toys (toy_id, toy_name, colour)
values ( 10, 'Blue Dinosaur', 'blue' );

/* TODO */
rollback to savepoint to_nine;

select * from toys
where toy_id IN ( 8, 9, 10 );

/* TODO */
rollback;

select * from toys
where toy_id in ( 8, 9, 10 );
```

## 8. Multi-table Insert
You can also insert rows into many tables in one statement. This is a multi-table insert. The general format of it is:

```sql
insert all
  into tab1 ( col1, col2, ...) values ( 'val1', 'val2', ...)
  into tab2 ( col1, col2, ...) values ( 'val1', 'val2', ...)
  into ...
  select * from query;
```

Like a multi-row insert, the source of these values must be a query. It adds a row to all the tables you list. You can even insert to the same table more than once!

The query below uses the dual table to select the value 0. Then insert it to bricks twice and toys once:

```sql
insert all
	into bricks ( brick_id ) values ( id )
	into bricks ( brick_id ) values ( id )
	into toys ( toy_id ) values ( id )
select 0 id from dual;

select * from toys
where toy_id = 0;

select * from bricks
where brick_id = 0;
```

Dual is a special one-row table in Oracle Database. You can use this to select values or functions not stored in a real table.

## 9. Conditional Multi-table Insert
When doing a multi-table insert, you may want choose which tables to add rows to at runtime. You can do this by adding a when clause before "into table". The database will only add rows to this table if the condition is true.

Conditional multi-table inserts come in two forms: all and first.

```sql
insert [ all | first ]
  when test1 then 
    into t1 ...
  when test2 then 
    into t2 ...
    into t3 ...
  else
    into t4 ...
  select * from query;
```

### All

When you specify all, the database evaluates every condition for each row the query returns. Each row goes in every table where the condition is true.

You can also add an else clause. This inserts rows which match no other conditions.

### First

Insert first evaluates the conditions from top to bottom. If a row matches many conditions, it only adds the row to the highest one that is true. It skips all remaining tests.

As with insert all, you can provide an else clause to catch rows which match no other conditions.

You can see the difference between all and first in code below. The row for toy_id 11 has the name Cuteasaurus and colour blue. So it matches both of these tests:

```sql
  when colour = 'blue' then
    ..
  when toy_name = 'Cuteasaurus' then
```

So insert all adds two copies of this row into bricks. But insert first stops processing when identifies the colour as blue. So it only inserts one copy:

```sql
insert into toys (toy_id, toy_name, colour) values (11, 'Cuteasaurus', 'blue');
insert into toys (toy_id, toy_name, colour) values (12, 'Sir Stripypants', 'blue');
insert into toys (toy_id, toy_name, colour) values (13, 'White Rabbit', 'white');

exec savepoint post_toys;

insert all
    when colour = 'blue' then
    	into bricks (brick_id, colour) 
    	values (toy_id, colour)
	when toy_name = 'Cuteasaurus' then
		into bricks (brick_id, colour) 
    	values (toy_id, colour)
	else
		into bricks (brick_id, colour) 
    	values (toy_id, colour)
	select * from toys
	where toy_id >= 11;

select * from bricks where brick_id >= 11;

rollback to savepoint post_toys;

insert first
	when colour = 'blue' then
		into bricks (brick_id, colour) 
    	values (toy_id, colour)
	when toy_name = 'Cuteasaurus' then
		into bricks (brick_id, colour) 
    	values (toy_id, colour)
	else
		into bricks (brick_id, colour) 
    	values (toy_id, colour)
	select * from toys
	where toy_id >= 11;

select * from bricks
where brick_id >= 11;

rollback;
```
