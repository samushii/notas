### SQL Fundamentals

> The relationship between tables is called schema.


Interacting with MySQL/MariaDB:

```
mysql -u <user> -h <host> -P <port> -p (you will be prompted for the password)
```


Useful commands:

> In-line queries should always be terminated with a semi-colon ( ; ).
 
```sql
SHOW DATABASES;

USE <database-name>;

SHOW TABLES;

DESCRIBE <table-name>;

SELECT * FROM <table-name>;

SELECT * FROM <table-name> ORDER BY <column-name> [ASC|DESC] [, <column-name> [ASC|DESC] ...];

SELECT * FROM <table-name> LIMIT <offset>, <limit-count>;

SELECT * FROM <table-name> WHERE <condition>;

SELECT * FROM <table-name> WHERE username LIKE '_%'; 

	ps.: % matches any character (0 or more occurrences); _ matches any character (exactly 1 occurrence).



UPDATE <table-name> SET password = new_pass WHERE username = user;
```

> AND, OR and NOT logical operators can be used (also can be used as symbols: &&, || and ! respectively).










