# Delete & Truncate: Removing Rows

### Content

1. Introduction
2. Delete
3. Truncate
4. Soft Deletes : Views, VPD, In-Database Archiving

-----------------------------------------------------------------------------------------------------------------------
### Prerequisite SQL
```sql
create table toys (
    toy_name varchar2(30),
    price    number(4, 2)    
);
insert into toys values ('Baby Turtle', 0.01);
insert into toys values ('Miss Snuggles', 0.51);
insert into toys values ('Cuteasaurus', 10.01);
insert into toys values ('Sir Stripypants', 14.03);
insert into toys values ('Purple Ninja', 14.22);

commit;
```

## 1. Introduction
This tutorial teaches you how to remove rows from a table using delete and truncate. It uses the following table to show how these work:

```sql
select * from toys;
```

## 2. Delete
Delete is another form of DML statement. You use it to remove rows from a table. An unqualified delete will erase all the rows from the table. So toys is empty after the delete statement:

```sql
delete from toys;

select * from toys;

rollback;

select * from toys;
```

The rollback undoes the delete, restoring the rows.

As with insert and update, you can supply a where clause. This only removes rows where the clause is true. So the following deletes the rows where the prices is less than one:

```sql
delete toys where price < 1;

select * from toys;

rollback;

select * from toys;
```

Note that the from clause is optional in delete.

Like update, delete locks the affected rows. This is from the time the statement starts until you commit or rollback the transaction. If someone else tries to update rows you're deleting, you block their update until your transaction ends.

### Try it
Complete the following statement to remove the row for the toy_name Purple Ninja:

```sql
select * from toys;

delete from toys
where toy_name = 'Purple Ninja';

select * from toys;
rollback;
```

## 3. Truncate
You can also remove all the rows from a table with truncate. Unlike delete, this is a Data Definition Language (DDL) statement in Oracle Database.

DDL statements (create table, alter table, etc.) commit. So you can't rollback a truncate!

Note how, even after the rollback, the table remains empty:

```sql
select * from toys;

truncate table toys;

select * from toys;

rollback;

select * from toys;
```

Truncate is also an all-or-nothing statement. You can't remove some rows using a where clause. If you add one, the statement throws an error:

```sql
truncate table toys where price < 1;
```

So with these gotchas, you may be wondering why you'd use truncate over delete. One key reason: performance.

Truncate is fast. It marks the table as empty without touching the data. So truncate operations are instant, no matter how many rows the table stores.

Whereas delete processes each row. So as the number of rows in the table increases, the time to delete them all will also increase. Deleting millions of rows can take hours to complete!

Truncate also deallocates a table's storage. But the space remains allocated to the table with delete.

If you want the space to stay assigned to the table after a truncate, use the "reuse storage" clause, like so:

```sql
truncate table toys reuse storage;
```

Keeping the storage is useful if you re-insert a similar number of rows soon after the truncate. For example, on a daily load from an external system. Leaving the space assigned to the table reduces the work the database does during the load.

## 4. Soft Deletes
Once you commit a delete or run a truncate, the rows are gone. To recover the data you need to restore from a backup*.

*Oracle Flashback technologies enable you to [recover lost data without a backup!](https://blogs.oracle.com/sql/post/how-to-recover-data-without-a-backup)

This is time-consuming and awkward. So it's hard to recover data deleted by accident. Many businesses are also subject to regulations stating that they can't remove certain types of data. For example, financial transactions.

So many applications implement a "soft delete" instead. This adds an "is deleted" flag to your tables. For example:

```sql
alter table toys add is_deleted varchar2(1) default 'N';
```

When adding new rows, ensure this value is N (No):

```sql
delete toys;

insert into toys values ('Baby Turtle', 0.01, 'N');
insert into toys values ('Miss Snuggles', 0.51, 'N');
insert into toys values ('Cuteasaurus', 10.01, 'N');
insert into toys values ('Sir Stripypants', 14.03, 'N');
insert into toys values ('Purple Ninja', 14.22, 'N');

select * from toys;

commit;
```

Now, to "delete" rows, you run an update. This sets the deleted flag to Yes:

```sql
update toys
set is_deleted = 'Y'
where toy_name = 'Cuteasaurus';

select * from toys;
```

But now you need to filter out the "deleted" values in most queries. This makes your code more complicated. To get only active rows, you need to add a where clause to all these queries. For example:

```sql
select * from toys
where is_deleted = 'N';
```

Luckily Oracle Database offers many ways to simplify this, including:

- Using views
- Virtual Private Database (VPD)
- In-Database Archiving

### 4.1 Views
The most universal way is to create a view over the top of the table. This contains the query excluding "deleted" rows. You change your application to query the view instead of the table.

For example:
```sql
create or replace view active_toys as
	select * from toys
	where is_deleted = 'N';

select * from active_toys;
```

### 4.2. VPD
You use VPD to control which users can see which rows. It does this by adding where clauses to your queries based on policies. This is primarily a security feature, allowing you to stop people seeing sensitive data without clearance. But you can also use it to manage soft-deletion.

For more on VPD, [read this article](https://oracle-base.com/articles/8i/virtual-private-databases).

### 4.3. In-Database Archiving
Oracle Database 12c introduced In-Database Archiving. This offers a new way to show or hide removed rows. It adds the invisible column ora_archive_state to each table you enable it for.

Do this with the following command:

```sql
alter table toys row archival;
```

You then "delete" rows by setting ora_archive_state to any value other than zero. For example:

```sql
update toys
set    ora_archive_state = '1'
where  toy_name = 'Baby Turtle';

select * from toys;
```

You control which rows are visible in the session. If you set the row archival visibility to all, you see everything:

```sql
select * from toys;

alter session set row archival visibility = all;

select * from toys;
```

To auto-filter out the deleted rows, set the visibility to active:

```sql
select * from toys;

alter session set row archival visibility = active;

select * from toys;
```
