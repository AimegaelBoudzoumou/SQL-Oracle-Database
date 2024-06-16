# Columns and Data Types

## 1. Defining Columns

A table in Oracle Database can have up to 1,000 columns. You define these when you create a table. You can also add them to existing tables.

Every column has a data type. The data type determines the values you can store in the column and the operations you can do on it. The following statement creates a table with three columns. One varchar2, one number, and one date:

```sql
create table this_table_has_three_columns (
  this_is_a_character_column varchar2(100),
  this_is_a_number_column    number,
  this_is_a_date_column      date
);
```

## 2. Viewing Column Information

You can find details about the columns in your user's tables by querying user_tab_columns.

This query finds the name, type, and limits of the columns in this schema:

```sql
select table_name, column_name, data_type, data_length, data_precision, data_scale
from user_tab_columns;
```

### Try It!
Complete the following statement to create the toys table with a column called toy_name:

```sql
create table toys (
  /* TODO */ varchar2(10)
);
```

This is a solution :

```sql
create table toys (
    toy_name varchar2(10)
);

select table_name, column_name, data_type, data_length
from   user_tab_columns
where  table_name = 'TOYS';
```

## 3. Data Types

### Character Data Types
Oracle Database has three key character types:
- varchar2 : This stores variable length text. You need to specify an upper limit for the size of these strings. In Oracle Database 11.2 and before, the maximum you can specify is 4,000 bytes. From 12.1 you can increase this length to 32,767.
- char : These store fixed-length strings. If the text you insert is shorter than the max length for the column, the database right pads it with spaces. /
The maximum size of char is 2,000 bytes.
- clob : If you need to store text larger than the upper limit of a varchar2, use a clob. This is a character large object. It can store data up to (4 gigabytes - 1) * (database block size). In a default Oracle Database installation this is 32Tb!

Note : Each of these types also has an N variation; nchar, nvarchar2, and nclob. These store Unicode-only data. It's rare you'll use these data types.

### Numeric Data Types
The built-in numeric data types for Oracle Database are:

