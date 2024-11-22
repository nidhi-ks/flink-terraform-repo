# Workshop Instructions

## . Creating Tables in Flink

### 1. View the `orders_data` table:

Run the following command to view the table structure:

```sql
SHOW CREATE TABLE orders_data;
```
Shows the DDL statement for the table 

Output : 

```sql
CREATE TABLE `orders_data` (
  `key` VARBINARY(2147483647),
  `ordertime` BIGINT NOT NULL,
  `orderid` INT NOT NULL,
  `itemid` VARCHAR(2147483647) NOT NULL,
  `orderunits` DOUBLE NOT NULL,
  `address` ROW<`city` VARCHAR(2147483647) NOT NULL, `state` VARCHAR(2147483647) NOT NULL, `zipcode` BIGINT NOT NULL> NOT NULL
)
DISTRIBUTED BY HASH(`key`) INTO 3 BUCKETS
WITH (
  'changelog.mode' = 'append',
  'connector' = 'confluent',
  'kafka.cleanup-policy' = 'delete',
  'kafka.max-message-size' = '2097164 bytes',
  'kafka.retention.size' = '0 bytes',
  'kafka.retention.time' = '7 d',
  'key.format' = 'raw',
  'scan.bounded.mode' = 'unbounded',
  'scan.startup.mode' = 'earliest-offset',
  'value.format' = 'avro-registry'
)
```

### 2. Adding Data to `orders_topic`

```sql
CREATE TABLE `orders_topic` (
  `key` VARBINARY(2147483647),
  `ordertime` BIGINT NOT NULL,
  `orderid` INT NOT NULL,
  `itemid` VARCHAR(2147483647) NOT NULL,
  `orderunits` DOUBLE NOT NULL,
  `address` ROW<`city` VARCHAR(2147483647) NOT NULL, `state` VARCHAR(2147483647) NOT NULL, `zipcode` BIGINT NOT NULL> NOT NULL
);
```
Creates a table orders_topic

```sql
INSERT INTO orders_topic
SELECT * 
FROM orders_data;
```
Inserts data into orders_topic table from orders_data table 

## Filtering in Flink: Price and Quantity Greater Than 0


### 3. Creating the `filtered_orders` Table

```sql
CREATE TABLE `filtered_orders` (
  `key` VARBINARY(2147483647),
  `ordertime` BIGINT NOT NULL,
  `orderid` INT NOT NULL,
  `itemid` VARCHAR(2147483647) NOT NULL,
  `orderunits` DOUBLE NOT NULL,
  `address` ROW<`city` VARCHAR(2147483647) NOT NULL, `state` VARCHAR(2147483647) NOT NULL, `zipcode` BIGINT NOT NULL> NOT NULL
);
```
Creates table filteres_orders

```sql
Copy code
INSERT INTO `filtered_orders`
SELECT * 
FROM orders_topic
WHERE `orderunits`> 5.00
```
Inserts data into filtered_orders after filtering data based on conditions from trades_orders 


### 5. Running Aggregates and Window Functions

```sql
create table totalnumberoforders_per_address_10min (
window_Start timestamp(3) not null ,
window_end timestamp(3) not null ,
total_number_of_units_ordered double not null,
`address` ROW<`city` string NOT NULL, `state` string NOT NULL, `zipcode` BIGINT NOT NULL> NOT NULL
);
```
Creates a table totalnumberoforders_per_address_10min

```sql
INSERT INTO totalnumberoforders_per_address_10min SELECT 
    window_start, 
    window_end,  
    SUM(orderunits) AS total_number_of_units_ordered,
    address
FROM TABLE (
    TUMBLE(TABLE orders_topic, DESCRIPTOR($rowtime), INTERVAL '10' MINUTES)
)
GROUP BY 
    address, 
    window_start, 
    window_end;
```
The query groups orders by address into 10-minute windows, summing the orderunits in each window. It uses the TUMBLE function to create time-based partitions and calculates the total number of units ordered per address.






