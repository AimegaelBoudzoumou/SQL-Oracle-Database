# Tables

## Creating a Table
```sql
create table toys (
    toy_name varchar2(50),
    weight   number
);
```

```sql
create table bricks (
    colour varchar2(10),
    shape  varchar2(10)
);

```
## Viewing Table Information
```sql
select table_name, iot_name, iot_type, external, 
       partitioned, temporary, cluster_name
from   user_tables;
```
```SQL
select table_name
from   user_tables
where  table_name = 'BRICKS';
```

## Table Organization
Create table in Oracle Database has an organization clause. This defines how it physically stores rows in the table.\
The options for this are:\ 
- Heap
- Index
- External
By default, tables are heap-organized. This means the database is free to store rows wherever there is space. You can add the "organization heap" clause if you want to be explicit:

```sql
create table toys_heap (
    toy_name varchar2(100)
) organization heap;
```

## Index-Organized Tables
Unlike a heap table, an index-organized table (IOT) imposes order on the rows within it. It physically stores rows sorted by its primary key. To create an IOT, you need to:

- Specify a primary key for the table
- Add the organization index clause at the end

```sql
create table bricks_iot (
    bricks_id integer primary key
) organization index;
```
You can find IOT in the data dictionary by looking at the column IOT_TYPE. This will return IOT if the table is index-organized:

```sql
select table_name, iot_type
from user_tables
where table_name = 'BRICKS_IOT';
```








