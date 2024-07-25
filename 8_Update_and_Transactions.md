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

```sql
update bricks
set    /* TODO */;
```

This is a solution:
```sql
update bricks
set unit_weight = 21;

select unit_weight from bricks;
```

## 3. Filtering Updates
It's rare you want to update all the rows in a table. Usually you only want to change a few.

You can do this by providing a where clause. This works in the same way as in select statements. Only rows where the conditions are true will change.

This update changes the quantity to 1,422 for all rows with colour of red:

```sql
update bricks
set    quantity = 1422
where  colour = 'red';

select * from bricks
```
### Try it

Complete the update below to set the unit_weight of all the rows storing the shape cube to 5:

```sql
update bricks
set    /* TODO */
where  /* TODO */;
```

This is a solution:
```sql
update bricks
set unit_weight = 5
where shape = 'cube';
```

## 4. Transactions
A transaction is the smallest unit of work which leaves the database in a consistent state. When a user saves changes in your application, this may generate many inserts, updates or deletes in your code. All these changes form one transaction.

A transaction ends when you commit or rollback (to the previous commit). You should only commit after running all the changes.

For example, say you paint ten of the green cubes blue. So you need to increase the quantity of blue cubes by 10. And decrease the green bricks by the same amount. If one update completes but not the other the total quantity will be out by 10.

This is can happen if you place a commit between the updates, as in this code:

```sql
update bricks
set   quantity = quantity - 10
where colour = 'green'
and   shape = 'cube';

commit;

update bricks
set   quantity = quantity + 10
where colour = 'blue'
and   shape = 'cube';

commit;

select * from bricks;

If the update of the blue rows fails for some reason, you've removed ten green cubes, but not added ten blue. So these bricks are "missing". Resolving this in code is hard. To fix the error, it's likely you'll need to run a one-off update.

To avoid this problem, move the commit to the end:

```sql
update bricks
set   quantity = quantity - 10
where colour = 'green'
and   shape = 'cube';

update bricks
set   quantity = quantity + 10
where colour = 'blue'
and   shape = 'cube';

commit;

select * from bricks;
```
Now both updates are in the same transaction. If either fails, you can rollback the whole transaction and try again.


## 5. Deadlocks
When a transaction runs many updates, you need to take care. If two or more people try to change the same rows at the same time, you can get into deadlock situations. This is where sessions form a circle, each waiting on the other.

In the example below, both transactions start by updating different rows. So the first session has locked the rows with the colour red. And the second locks rows with the colour blue.

The first session then tries to update rows storing blue. But transaction two has these locked! So it's blocked, waiting for transaction 2 to complete.

But transaction two, instead of committing or rolling back, tries to update rows storing red. But transaction one has these rows locked! So it's stuck. At this point both sessions are waiting for the other to release a lock.

Deadlock!

![image](https://github.com/user-attachments/assets/550cc8ac-bee7-4659-8aa9-aa295b5462a0)

When deadlock happens, Oracle Database will detect it. The database will then stop one of the statements, raising an ORA-00060.

You can simulate a deadlock in one session with autonomous transactions. These start a transaction within a transaction.

So the following code contains two transactions. The parent transaction updates all the rows in bricks, blocking other changes to these rows. The autonomous transaction after it also tries to update all the rows.

This is impossible. For the parent transaction to continue, the child autonomous transaction must complete. But the update in the parent blocks the update in the child child!

So the database identifies this as a deadlock. It stops the second update, raising ORA-00060. And allows you to commit or rollback the first update:

```sql
update bricks
set    quantity = 60;

declare
	pragma autonomous_transaction;
begin
	update bricks
	set    unit_weight = 55;
	commit;
end;
/

select * from bricks;

rollback;
```
Note that an update locks the whole row. Even though the two updates set different columns, the second is still blocked. Leading to deadlock.

Deadlocks are caused by errors in your code. To avoid them, you must change how you write your transactions.

## 6. Select For Update
## 7. Lost Updates
## 8. Optimistic Locking

```sql
```
