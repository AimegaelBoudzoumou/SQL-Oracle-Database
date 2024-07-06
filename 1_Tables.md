# Tables

### Content

1. Creating a Table
2. Viewing Table Information
3. Table Organization
4. Temporary Tables
5. Partitioning Tables
6. Table Clusters
7. Dropping Tables

----------------------------------------------------------------------------------------------------------------------

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
The options for this are:
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

## 5. Partitioning Tables

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
create table toys_range (
    toy_name varchar2(100)
)   partition by range (toy_name) (
    partition p0 values less than ('b'),
    partition p1 values less than ('c')
);

create table toys_list (
    toy_name varchar2(100)    
)   partition by list (toy_name) (
    partition p0 values ('Sir Stripypants'),
    partition p1 values ('Miss Snuggles')
);

By default a partitioned table is heap-organized. But you can combine partitioning with some other properties. For example, you can have a partitioned IOT:

```sql
create table toys_part_iot (
    toy_id integer primary key,
    toy_name varchar2(100)
)   organization index
    partition by hash (toy_id) partitions 4;
````

The database sets the partitioned column of *_tables to YES if the table is partitioned. You can view details about the partitions in the *_tab_partitions tables:

```sql
select table_name, partitioned
from user_tables
where table_name in ( 'TOYS_HASH', 'TOYS_LIST', 'TOYS_RANGE', 'TOYS_PART_IOT' );

select table_name, partition_name
from user_tab_partitions;

```

Note that partitioning is a separately licensable option of Oracle Database. Ensure you have this option before using it!

### Try It!
Complete the following statement to create a hash-partitioned table. This should be partitioned on brick_id and have 8 partitions:

```sql
create table bricks_hash (
  brick_id integer
) partition by /*TODO*/;
```
__This is the solution :__
```sql
create table bricks_hash (
  brick_id integer
) partition by hash (brick_id) partitions 8;

select table_name, partitioned
from   user_tables
where 	table_name = 'BRICKS_HASH';
```

## 6. Table Clusters
A table cluster can store rows from many tables in the same physical location. To do this, first you must create the cluster:

```sql
create cluster toy_cluster (
    toy_name varchar2(100)
);
```

Then place your tables in it using the cluster clause of create table:

```sql
create table toys_cluster_tab (
    toy_name varchar2(100)
)   cluster toy_cluster ( toy_name );

create table toy_owners_cluster_tab (
    owner    varchar2(100),
    toy_name varchar2(100) 
)   cluster toy_cluster ( toy_name );
```

Rows that have the same value for toy_name in toys_clus_tab and toy_owners_clus_tab will be in the same place. This can make it faster to get a row for a given toy_name from both tables.

You can view details of clusters by querying the *_clusters views. If a table is in a cluster, cluster_name of *_tables tells you which cluster it is in:

```sql
select cluster_name from user_clusters;

select table_name, cluster_name
from user_tables
where table_name in ( 'TOYS_CLUSTER_TAB', 'TOY_OWNERS_CLUSTER_TAB' );
```

__Note:__ Clustering tables is an advanced topic. They have some restrictions. So make sure you read up on these before you use them!

## 7. Dropping Tables
You can remove existing tables with the drop table command. Just add the name of the table you want to destroy:

```sql
create table toys_heap (
    table_name varchar2(100)
)   organization heap;

select table_name
from   user_tables
where  table_name = 'TOYS_HEAP';

drop table toys_heap;

select table_name
from toys_heap
where table_name = 'TOYS_HEAP';
```

