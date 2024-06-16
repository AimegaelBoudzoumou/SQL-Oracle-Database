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
- __varchar2__ : 4,000 bytes (Oracle Database 11.2 and before) and 32,767 (from 12.1).
- __char__ : 2,000 bytes.
- __clob__ : It can store data up to (4 gigabytes - 1) * (database block size). In a default Oracle Database installation this is 32Tb!

__Note :__ Each of these types also has an N variation; nchar, nvarchar2, and nclob. These store Unicode-only data. It's rare you'll use these data types.

```sql
create table character_data (
    varchar_10_col   varchar2(10),
    varchar_4000_col varchar2(4000),
    char_10_col      char(10),
    clob_col         clob
);

select column_name, data_type, data_length
from user_tab_columns
where table_name = 'CHARACTER_DATA';
```

### Numeric Data Types
The built-in numeric data types for Oracle Database are:
- numeric
- float
- binary_float
- binary_double

You use these to store numeric values, such as prices, weights, etc.

```sql
create table numeric_data (
    number_3_sf_2_dp  number(3, 2), -- sf sf for significant, dp for decimal point
    number_3_sf_2     number(3, -2),
    number_5_sf_0_dp  number(5, 0),
    integer_col       integer,
    float_col         float(10),
    real_col          real,
    binary_float_col  binary_float,
    binary_double_col binary_double
);

select column_name, data_type, data_length, data_precision, data_scale
from user_tab_columns
where table_name = 'NUMERIC_DATA';
```

__Note :__ sf sf for significant, dp for decimal point
