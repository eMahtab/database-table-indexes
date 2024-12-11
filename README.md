# Database Table Indexes

## Database Setup : 

The `test` database contains two tables `users` and `messages`

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

### Indexes in users and messages tables:
Below is the output from **`SHOW INDEX FROM users`** and **`SHOW INDEX FROM messages`**

!["users and messages table indexes"](table-indexes.png?raw=true)

As we can see from above screenshot, **`users` table have one index, which is Primary index on `id` column**

And **`messages` table have one Primary index (on column `id`) and two Secondary indexes (one of column `sender_id` and other on column `recipient_id`)**
