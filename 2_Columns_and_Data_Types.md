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

### 3.1. Character Data Types
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

### 3.2. Numeric Data Types
The built-in numeric data types for Oracle Database are:
- numeric
- float
- binary_float
- binary_double

You use these to store numeric values, such as prices, weights, etc.

```sql
create table numeric_data (
    number_3_sf_2_dp  number(3, 2), -- sf for significant, dp for decimal point
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

__Note :__ sf for significant figures, dp for decimal point

### 3.3. Datetime and Interval Data Types

Oracle Database has the following datetime data types:

- date
- timestamp
- timestamp with time zone
- timestamp with local time zone

Examples :

date'2018-02-14'

to_date ( '2018-07-23 09:00 AM', 'YYYY-MM-DD HH:MI AM' ) # If you need to state the time of day too, you need to use to_date

timestamp '2018-02-14 09:00:00.123' # If you need greater precision than dates, use timestamps

to_timestamp ( '2018-07-23 09:00:00.123 AM', 'YYYY-MM-DD HH:MI:SS.FF AM' )

__Timestamp :__ You can store time durations with intervals. Oracle Database has two interval types: year to month and day to second.

```sql
create table datetime_data (
  date_col                      date,
  timestamp_with_3_frac_sec_col timestamp(3),
  timestamp_with_tz             timestamp with time zone,
  timestamp_with_local_tz       timestamp with local time zone,
  year_to_month_col             interval year to month,
  day_to_second_col             interval day to second
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'DATETIME_DATA';
```

### 3.2. Binary Data Types

You use binary data to store in its original format. These are usually other files, such as graphics, sound, video or Word documents. There are two key binary types: raw and blob.

__Raw__
Like with character data, raw is for smaller items. You specify the maximum length of data for each column. It has a maximum limit of 2,000 bytes up to 11.2 and 32,767 from 12.1.

__Blob__
Blob stands for binary large object. As with clob, the maximum size you can store is (4 gigabytes - 1) * (database block size).

The following creates a table with binary data type columns:

```sql
create table binary_data (
    raw_col raw(1000),
    blob_col blob
);

select column_name, data_type, data_length, data_precision, data_scale
from user_tab_columns
where table_name = 'BINARY_DATA';
```

### Try It!
Complete the following statement to create a table with the following columns:

brick_id of type number(20, 0)
colour of type varchar2, max size 10
price of type number with precision 10 and scale 2
purchased_date of type date

```sql
create table bricks (
  /* TODO */
);
```

This is a solution :

```sql
create table bricks (
    brick_id number(20,0),
    coulor varchar2(10),
    price number(10,2),
    purchases_date date
);

select column_name, data_type, data_length, data_precision, data_scale
from   user_tab_columns
where  table_name = 'BRICKS';
```



