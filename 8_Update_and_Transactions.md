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

## 3. Filtering Updates

## 4. Transactions
## 5. Deadlocks
## 6. Select For Update
## 7. Lost Updates
## 8. Optimistic Locking

```sql
```
