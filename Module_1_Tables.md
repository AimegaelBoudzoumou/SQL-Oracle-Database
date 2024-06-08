# Tables

## Creating a Table
```sql
create table toys (
    toy_name varchar2(50),
    weight   number
);

create table bricks (
    colour varchar2(10),
    shape  varchar2(10)
);

```



## Viewing Table Information
```sql
select table_name, iot_name, iot_type, external, 
	partitioned, temporary, cluster_name
from user_tables;
```
