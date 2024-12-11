# Database Table Indexes

## Database Setup : 

The `test` database contains two tables `users` and `messages`, the tables and records in the tables are created by importing the (https://github.com/eMahtab/mysql-test-dataset/blob/main/users-and-messages/test_database.zip) database dump.

Its medium sized database having in total 110 Million records, so **importing the database dump will take a long time**.

!["database dump import"](data-import-successful.png?raw=true)

### Dataset Size :

**users table = 10 Million records**

**messages table = 100 Million records**

!["users and messages table in test database"](tables.png?raw=true)

#### Schema  : 
**The tables were originally created using below DDL statements :**

`users` table have id (which denotes user id) as Primary Key, and `messages` table also have id (which denotes message id) as Primary Key. 

`messages` table have columns `sender_id` and `recipient_id` which are Foreign Key, referencing `id` column of `users` table

```sql
CREATE TABLE users (
    id BIGINT,
    name VARCHAR(50),
    username VARCHAR(30),
    PRIMARY KEY (id)
);

CREATE TABLE messages (
    id BIGINT,
    sender_id BIGINT,
    recipient_id BIGINT,
    message TEXT,
    created_at DATETIME NOT NULL,
    edited_at DATETIME DEFAULT NULL,
    deleted_at DATETIME DEFAULT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (recipient_id) REFERENCES users(id)
);
```

!["users and messages table records"](table-records.png?raw=true)

## Indexes in users and messages tables:

Below is the output from **`SHOW INDEX FROM users`** and **`SHOW INDEX FROM messages`**

!["users and messages table indexes"](table-indexes.png?raw=true)

As we can see from above screenshot, **`users` table have one index, which is Primary index on `id` column**

And **`messages` table have one Primary index (on column `id`) and two Secondary indexes (one on column `sender_id` and other one on column `recipient_id`)**

## Primary and Secondary Index :

A Primary index is always unique, and it uniquely identifies each row in the table and enforces uniqueness on the primary key column(s). A table can have only one primary index.

A Secondary index may or may not be unique, secondary indexes are created to optimize query performance for columns not covered by the primary index. A table can have multiple secondary indexes.

If you see the `CREATE TABLE` command above for `messages` table, we did not explicitly created secondary indexes, it was automatically created by MySQL for enforcing referential integrity **efficiently**. Operations like `ON DELETE CASCADE` or `ON UPDATE CASCADE` need to locate and modify rows in the referencing table quickly.

You can always check the indexes in a table using `SHOW INDEX FROM` command, to get more details.


# Always Check your query : EXPLAIN ANALYZE

## Fast query : using the Primary index in messages table (index lookup)
```sql 
select * from messages where id = 679802;
```
!["select message with id"](select-message-with-id.png?raw=true)

## Fast query : using the Secondary index in messages table (index lookup)
```sql 
select count(*) from messages where sender_id = 6452;
```
!["count messages from sender_id"](count-message-from-sender_id.png?raw=true)

## Fast query : using the Secondary index in messages table (index lookup and filter)
```sql 
select * from messages
where sender_id = 12098 AND 
created_at BETWEEN '2024-12-05 00:00:00' AND '2024-12-05 23:59:59';
```
!["index lookup and sort"](index-lookup-and-filter.png?raw=true)

## Fast query : using the Secondary index in messages table (index lookup and sort)
```sql 
select * from messages where sender_id = 44887 order by created_at ASC;
```
!["index lookup and sort"](index-lookup-and-sort.png?raw=true)

## Fast query : using the Secondary index in messages table (index lookup and filter and sort)
```sql 
select * from messages 
where sender_id = 889000 AND 
created_at BETWEEN '2024-12-01 00:00:00' AND '2024-12-07 23:59:59'
order by created_at ASC;
```
!["index lookup and filter and sort"](index-lookup-filter-and-sort.png?raw=true)

## Fast query : using multiple indexes in messages table 
```sql 
 select * from messages where sender_id = 8 AND recipient_id = 859523;
```
!["query using multiple indexes"](using-multiple-indexes.png?raw=true)

## Slow/Very Slow query : Table Scan (queried column doesn't have index)
If you query on a column which don't have index on it, then the query will be slow or very slow or will take unacceptable amount of time, 
depending on how many records that table have. A Table scan is costly and should be avoided.
One of the way to solve this issue, is to create an index on the column on which we want to query.

```sql 
select count(*) from messages
where created_at BETWEEN '2024-12-03 00:00:00' AND '2024-12-03 23:59:59';
```
!["table scan"](table-scan-created_at.png?raw=true)

# Table Indexes and Index Size

Three indexes on `messages` table take up around 7.4 GB space.

To get the total size of index and actual data stored inside a table, use below sql command

```sql
SELECT TABLE_NAME,
       DATA_LENGTH / (1024 * 1024 * 1024) AS DataSize_GB,
       INDEX_LENGTH / (1024 * 1024 * 1024) AS IndexSize_GB,
       (DATA_LENGTH + INDEX_LENGTH) / (1024 * 1024 * 1024) AS TotalSize_GB
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 'messages';
```
!["table indexes with size"](table-indexes-with-size.png?raw=true)

# Creating index on created_at column

!["creating index on created_at column"](index-on-created_at.png?raw=true)

# Query performance improvement after creating index (from over 90 seconds to under 6 seconds)

!["Query performance improvement"](query-performance-improvement.png?raw=true)

# Indexing Overhead :
Indexing comes with a small cost, whenever a new row inserted or a row deleted, the indexes needs to updated to reflect the updated state of database table.
Similarly when a row is updated (which updates the indexed column), then also index needs to be updated.

So indexing does adds to slowing down the write operation.

