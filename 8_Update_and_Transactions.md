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
You're at risk of deadlock if a transaction runs two or more update statements on the same table. And two people running the transaction may update the same rows, but in a different order.

In these cases you should lock the rows before running any updates. You can do this with select for update. Like update itself, this locks rows matching the where clause. To use is, place the "for update" clause after your query.

So the following locks all the rows with the colour red:

```sql
select * from bricks
where colour = 'red'
for   update;
```

No one else can update, delete or select for update these rows until you commit or rollback.

You can use this to avoid deadlock by running a select for update at the start of your transaction. For example, to avoid the deadlock described in the previous module, a run select for update at the start. This selects rows for the colours red and blue. Then run the updates:

```sql
select * from bricks
where colour IN ( 'red', 'blue' )
for   update;

update bricks
set    quantity = 1001
where  colour = 'red';

update bricks
set    quantity = 1722
where  colour = 'blue';
```

Changing the transactions to do this gives the following outcome:

![image](https://github.com/user-attachments/assets/73db8e9a-2f0c-4ab0-a9d5-02942e249790)

Deadlock can also happen when a transaction updates two or more different tables. If two separate transactions update two tables in a different order, you'll have deadlock.

This is harder to avoid. You need to ensure all transactions lock tables in the same order. You need define an order for acquiring locks. And ensure all developers on the application using the same method. This requires thorough documentation, code reviews, and testing to ensure deadlock is impossible.

## 7. Lost Updates
A lost update happens when two people change the same row and one overwrites another's changes. This is a common problem if you set values for all column not in the where clause. Regardless of whether the user changed the value.

For example, say currently there are 60 red cylinders in stock, with a unit weight of 13. Run this update to set these values:
```sql
update bricks
set    quantity = 60,
       unit_weight = 13
where  colour = 'red'
and    shape = 'cylinder';
```

You have two people managing brick stock. One handles quantities, the other weights. They both load the current details for red cylinders to the edit form using this query:

```sql
select *
from   bricks
where  colour = 'red'
and    shape = 'cylinder';
```
At this point they both see a quantity of 60 and weight of 13.

Then one person sets the weight to 8. And the other the quantity to 1,001. But they leave the value for the other attribute the same. They then both save their changes. The application updates both columns each time.

For each person, the update runs with the original value for the unchanged column. So the update to change the weight runs with these values:

```sql
update bricks
set    quantity = 60,   -- original quantity
       unit_weight = 8 -- new weight
where colour = 'red'
and   shape  = 'cylinder';

select * 
from bricks
where colour = 'red'
and   shape  = 'cylinder';
```

And the quantity change runs with these values:

```sql
update bricks
set    quantity = 1001,   -- new quantity
       unit_weight = 13   -- original weight
where colour = 'red'
and   shape  = 'cylinder';

select * 
from bricks
where colour = 'red'
and   shape  = 'cylinder';
```

As you can see, the second update sets the unit_weight from the new value of 8 back to the original, 13. So the update setting the weight to 8 is lost.

This problem can be hard to spot. Immediately after the person setting the weight saves their changes, everything looks fine. So they go on to do something else. Same for the stock level manager. So it may take some time for staff to realise there's a data error.

## 8. Optimistic Locking
You could avoid the issue above by having separate forms for managing weights and quantities. Saving changes in each of these runs an update which only changes values in the relevant column.

But separation like this isn't always possible. And it's more work to develop. In general to avoid lost updates you can use pessimistic or optimistic locking.

Pessimistic locking locks rows from the time the user first queries them with select for update. The lock remains until they save their changes. To use it, you need a stateful application. This is rare in the web, where most applications are stateless. Most applications you work on will use optimistic locking.

Optimistic locking works by checking values the user read are still the same when they update the row. If they aren't the update does nothing.

There are many ways you can do this, including:

- Adding a version number to each row
- Adding a last updated timestamp to each row
- Checking that the current values match those queried when you run your update

For example, to avoid the lost update described in the previous module, you could change the updates to:

```sql
update bricks
set    quantity = 1001,
       unit_weight = 13
where  colour = 'red'
and    shape  = 'cylinder'
and    quantity = 60    -- original quantity
and    unit_weight = 13 -- orginal weight
;

select *
from bricks
where colour = 'red'
and   shape  = 'cylinder';
```
This method gets cumbersome when the update changes many columns. You need a large where clause to check all the original values!

An easier way is to add a version or timestamp column to the table. When you run an update, check it has the same value as read at query time. And change it during the update.

For example, this adds the column version_number to bricks:


```sql
alter table bricks add (version_number integer default 1);

select * from bricks;
```

The updates check this value in their where clause. And increment it in the set clause.

When they query the row, both users get it with a version number of 1. So this is the value the application passes back to the database during update.

The first update succeeds, increasing the version number to 2:

```sql
update bricks
set    quantity = 60,
       unit_weight = 8,
       version_number = version_number + 1
where  colour = 'red'
and    shape  = 'cylinder'
and    version_number = 1;

select *
from   bricks
where  colour = 'red'
and    shape  = 'cylinder';
```

So when the second runs, the condition

version_number = 1

is false. So the update does nothing:

```sql
update bricks
set    quantity = 1001,
       unit_weight = 13,
       version_number = version_number + 1
where  colour = 'red'
and    shape  = 'cylinder'
and    version_number = 1;

select *
from   bricks
where  colour = 'red'
and    shape  = 'cylinder';
```
When using optimistic locking, your code needs to check for "no change" updates. Then inform the user the data have changed since they last read it. And ask them to resubmit their changes.
