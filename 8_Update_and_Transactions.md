# Update & Transactions

### Content

1. Introduction
2. Update Statement
3. Filtering Updates
4. Transactions
5. Deadlocks
6. Select For Update
7. Lost Updates
8. Optimistic Locking

-----------------------------------------------------------------------------------------------------------------------
### Prerequisite SQL
```sql
create table bricks (
    colour      varchar2(10),
    shape       varchar2(10),
    quantity    integer,
    unit_weight integer
);

insert into bricks values ( 'red', 'cylinder', 1, 13 );
insert into bricks values ( 'blue', 'cube', 51, 8 );
insert into bricks values ( 'green', 'cube', 600, 8 );

commit;
```

## 1. Introduction
This tutorial teaches you how to change values using the update statement. Like select and insert, this is also a DML statement. The examples use the following table:
```sql
select * from bricks;
```

## 2. Update Statement
There are two core parts to an update:

The name of the table you're changing. This goes after update
The columns you're changing and the values you set them to. These form a comma-separated list in the set clause
So the general form of an update is:

```sql
update table
set    col1 = 'value1',
       col2 = 'value2',
       ...
```

For example, the following sets the quantity of all rows to 60:
```sql
update bricks
set    quantity = 60;

select * from bricks;
```

An update can change any number of columns from one up to the total in the table.

When you update a row, the database locks it. This stops other people from changing the row. The lock remains until you commit or rollback.

If many people try to update the same rows at the same time, only the first person to start the update can complete it. All others must wait until they release the lock. If your application will run many updates at the same time, it can come grinding to a halt!

### Try it
Complete the following update to set the unit_weight of all bricks to 21:

````sql

```

This is a solution:
````sql

```


## 3. Filtering Updates

## 4. Transactions
## 5. Deadlocks
## 6. Select For Update
## 7. Lost Updates
## 8. Optimistic Locking

```sql
```