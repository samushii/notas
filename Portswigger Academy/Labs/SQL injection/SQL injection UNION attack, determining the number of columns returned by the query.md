
## Description:

this lab contains a SQL injection vulnerability in the product category filter, the results are returned in the application's response, so it is possible to use a UNION attack to retrieve data from other tables.

the first step is to determine the number of columns that are being returned by the query by performing a SQL injection UNION attack that return an additional row containing null values

## Solution:

	' UNION SELECT 123,NULL,NULL--

simple union injection starting with 1 column and adding more to see if the application stops returning an error/returns a different response



