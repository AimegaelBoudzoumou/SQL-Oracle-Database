# Tables

## 1. Creating a Table
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
## 2. Viewing Table Information
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

## 3. Table Organization
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

## 3.1. Index-Organized Tables
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

## 3.2. External Tables
You use external tables to read non-database files on the database server. For example, comma-separated values (CSV) files. To do this, you need to:
- Create a directory pointing to the location of the file on the server
- Use the organization external clause
- State the directory and name of the file you want to read

```sql
create or replace directory toy_dir as '/path/to/file';

create table toys_ext (
  toy_name varchar2(100)
) organization external (
  default directory tmp
  location ('toys.csv')
);
```

_Note: LiveSQL doesn't support external tables._

When you query this table, it will read from the file:\
/path/to/file/toys.csv\
This file must be accessible to the database server. You cannot use external tables to read files on your machine!

## 4. Temporary Tables
Temporary tables store session specific data. Only the session that adds the rows can see them. This can be handy to store working data.\
There are two types of temporary table in Oracle Database: global and private.

### 4.1. Global Temporary Tables
To create a global temporary table add the clause "global temporary" between create and table. For example:
```sql
create global temporary table toys_gtt (
  toy_name varchar2(100)
);
```
_The definition of the temporary table is permanent. All users of the database can access it. But only your session can view rows you insert._

### 4.2. Private Temporary Tables
Starting in Oracle Database 18c, you can create private temporary tables. These tables are only visible in your session. Other sessions can't see the table! \
To create one use "private temporary" between create and table. You must also prefix the table name with ora$ptt_:
```sql
create private temporary table ora$ptt_toys (
    toy_name varchar2(100)
);
```
### 4.3. Viewing Temporary Table Details
The column temporary in the *_tables views tell you which tables are temporary:
```sql
select table_name, temporary
from user_tables
where table_name in ('TOYS_GTT', 'ORA$PTT_TOYS');

select table_name, temporary
from user_tables;
```

Note that you can only see a row for the global temporary table. The database doesn't write private temporary tables to the data dictionary!

## Partitioning Tables

Partitioning logically splits up a table into smaller tables according to the partition column(s). So rows with the same partition key are stored in the same physical location.

There are three types of partitioning available:

- Range
- List
- Hash

To create a partitioned table, you need to:
- Choose a partition method
- State the partition columns
- Define the initial partitions

The following statements create one table for each partitioning type:

```sql

