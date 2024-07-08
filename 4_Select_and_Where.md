# Select and Where

### Content

1. Selecting Rows
2. Filtering Data
3. Combining Criteria
4. Lists of Values
5. Ranges of Values
6. Wildcards
7. Null
8. Negation

-----------------------------------------------------------------------------------------------------------------------

### Prerequisite SQL

```sql
create table toys (
    toy_name varchar2(100),
    colour   varchar2(100),
    price    number(10, 2)
);

insert into toys values ('Sir Stripypants', 'red', 0.01);
insert into toys values ('Miss Smelly_bottom', 'blue', 6.00);
insert into toys values ('Cuteasaurus', 'blue', 17.22);
insert into toys values ('MR Bunnikins', 'red', 14.22);
insert into toys values ('Baby Tutle', 'green', null);
```

## 1. Selecting Rows

```sql
select * from toys;
```

This query returns the price and colour columns of the toys, displaying price first:
```sql
select price, colour from toys;
```

## 2. Filtering Data
This gets the row that stores "Sir Stripypants" in the column toy_name:

```sql
select * from toys
where toy_name = 'Sir Stripypants';
```

```sql
select * from toys
where colour  = 'red';
```

## 3. Combining Criteria
You can combine many filters with AND & OR.

### AND
The following searches for rows where the toy_name is Sir Stripypants. And the colour is green. Sir Stripypants is red, so this query returns nothing:

```sql
select * from toys
where toy_name = 'Sir stripypants'
and colour = 'green';
```

### OR
OR fetches all the rows where either of the criteria are true. Baby Turtle has the colour green. So this query gets the row for them and Sir Stripypants:

```sql
select * from toys
where toy_name = 'ir Stripypants' or
      colour = 'green';
```

### Order of Precedence
AND has higher priority than OR. So if you include both in a where clause, the order you place them affects the results.

For example, the following two queries search for the same values from each column. But they return different rows:

```sql
select * from toys
where toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turte'
and   colour = 'green';

select * from toys
where colour = 'green'
and   toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle';
```

To avoid confusion in queries combining AND with OR, use parentheses. The database processes conditions inside the brackets first. So the following two queries both search for rows where:

- The toy_name is "Mr Bunnykins" or "Baby Turtle"
- And the colour is green

```sql
select * from toys
where ( toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle' )
and colour = 'green';

select * from toys
where colour = 'green'
and ( toy_name = 'Mr Bunnykins' or toy_name = 'Baby Turtle' );
```


### Try It!
Complete the following query to find the rows that have:
- The toy_name Sir Stripypants or the colour blue
- And a price equal to 6


4. Lists of Values
5. Ranges of Values
6. Wildcards
7. Null
8. Negation

-------------------------------
```sql

```


